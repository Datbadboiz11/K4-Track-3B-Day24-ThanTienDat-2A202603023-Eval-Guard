# Blueprint CI/CD cho RAG và guardrails

Nhóm: Nhóm cá nhân - Thân Tiến Đạt

Người phụ trách: Thân Tiến Đạt - 2A202603023

Đây là **đề xuất CI/CD**, không phải workflow đã được triển khai/chạy.
Nguồn kết quả: artifacts/ragas_50q.json, judge_results.json, guard_results.json.
Run timestamp: 2026-10-12T10:07:28.918925+00:00; guard models: ["gpt-4o-mini"].
Models generation/evaluator/judge từ A/B: Generation: ["gpt-4o-mini-2024-07-18"] (từ answers_50q.json và judge_results.json); Evaluator: ["BAAI/bge-m3", "gpt-4o-mini"] (từ ragas_50q.json); Judge: ["gpt-4o-mini"] (từ judge_results.json); Guard: ["gpt-4o-mini"] (từ guard_results.json).

## Kiến trúc và failure action

Input → Presidio → NeMo input → RAG → NeMo output → response.
Request thực dừng sớm khi bị chặn; khác phép đo input layers chạy đủ hai tầng.

### Luồng dữ liệu và cơ chế xử lý lỗi (Data Flow & Failure Action):
1. **Tầng 1 - Presidio PII Scanning**: Kiểm tra các mẫu dữ liệu định danh cá nhân nhạy cảm (`VN_CCCD`, `VN_PHONE`, `EMAIL_ADDRESS`, `PHONE_NUMBER`). Nếu phát hiện PII xâm nhập hoặc cố ý trích xuất PII, request bị short-circuit và từ chối lập tức với thông báo bảo mật chuẩn hóa. Nếu văn bản chỉ cần che giấu thông tin hiển thị (anonymization), dữ liệu được thay thế bằng thẻ phân loại `<TYPE>` trước khi tiếp tục.
2. **Tầng 2 - NeMo Input Rails**: Sử dụng LLM guardrail với Colang 1.0 và prompt tự kiểm tra (`self_check_input`) để chặn jailbreak, prompt injection, đóng giả quyền hạn, và các câu hỏi ngoài phạm vi HR/chính sách bảo mật công ty. Khi phát hiện vi phạm, hệ thống dừng luồng xử lý và trả về thông điệp từ chối `bot refuse to respond`.
3. **Tầng 3 - RAG Retrieval & Generation Pipeline**: Tìm kiếm ngữ nghĩa kết hợp BM25 + Qdrant Vector Search (BGE-M3), rerank bằng FlashRank/BGE-reranker-v2-m3, và sinh câu trả lời bằng LLM generator (`gpt-4o-mini`).
4. **Tầng 4 - NeMo Output Rails**: Kiểm duyệt câu trả lời trước khi gửi tới người dùng cuối qua `self_check_output` nhằm ngăn chặn rò rỉ dữ liệu mật, mã độc hoặc câu trả lời không an toàn. Nếu phát hiện vi phạm, nội dung không an toàn sẽ bị thay thế hoàn toàn bằng câu từ chối chuẩn mực.
5. **Quản lý Secrets & An toàn Logging**: Toàn bộ secrets và API keys (`OPENAI_API_KEY`, Qdrant tokens) được lưu trữ an toàn trong Secret Manager/Biến môi trường CI, tuyệt đối không hardcode trong repo hay log ra file. Mọi log ghi nhận vi phạm chỉ lưu trữ metadata ẩn danh (request_id, timestamp, rule_violated, latency), không ghi log nguyên văn nội dung PII của người dùng.
6. **Điều kiện kích hoạt Fallback**: Nếu tầng LLM generator gặp sự cố mạng hoặc timeout, pipeline kích hoạt chế độ `extractive_fallback` trích xuất nguyên văn đoạn trích có điểm liên quan cao nhất từ context tài liệu để phục vụ người dùng mà không gây sập hệ thống.

## Phạm vi và số đo latency

Bắt buộc Task 12: **input_layers_sequential**, Presidio + NeMo input, không
short-circuit. Engines tạo trước đo; có hai warm-up thật không tính vào mẫu.
Percentile nội suy tuyến tính từ raw samples; tổng từ tổng từng sample.

Số mẫu timed: 20; warm-up: 2; phương pháp: linear.

| Layer / scope | P50 ms | P95 ms | P99 ms | Tình trạng |
|---|---|---|---|---|
| Presidio | 6.64 | 14.94 | 18.00 | Đạt mục tiêu riêng tầng PII local (<50ms) |
| NeMo input | 834.10 | 1041.06 | 1046.96 | Vượt ngân sách do độ trễ mạng gọi LLM API |
| Input guard sequential | 839.67 | 1047.16 | 1055.72 | Vượt mục tiêu tổng 500ms (latency_budget_ok: False) |
| RAG | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| NeMo output | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| Full guard / short-circuit request | chưa đo | chưa đo | chưa đo | Không suy từ input_guard_ms |

Input-layer budget target: 500ms; kết quả thực tế/budget_ok: P95 = 1047.16ms / budget_ok = False.

### Phân tích Bottleneck và Kế hoạch đo mở rộng:
- **Nguyên nhân chính (Bottleneck)**: Tầng Presidio chạy cục bộ bằng Regex + SpaCy NLP cho tốc độ cực nhanh với P95 chỉ 14.94ms. Điểm nghẽn duy nhất nằm ở NeMo input rail (P95 đạt 1041.06ms) do phải thực hiện một cuộc gọi HTTP round-trip ra máy chủ OpenAI API tại Mỹ.
- **Giải pháp tối ưu hóa Production**: Thay thế việc gọi OpenAI API từ xa cho self_check_input bằng mô hình phân loại cục bộ hoặc LLM nhỏ chạy on-premise (như Llama-3-8B-Instruct quantized, vLLM local engine hoặc mô hình DeBERTa sequence classification chuyên dụng), giúp kéo giảm P95 NeMo input xuống dưới 80-120ms, đảm bảo tổng độ trễ input guard < 150ms.
- **Kế hoạch đo lường các tầng chưa đo**: Đối với tầng RAG và NeMo Output, kế hoạch đo sẽ được thực hiện bằng cách chèn timer `time.perf_counter()` tại điểm bắt đầu và kết thúc của từng block trong pipeline, thực hiện 20 lượt mẫu timed sau 2 lượt warm-up trên cùng tập 8 câu hỏi kiểm soát.

## CI gates đề xuất

- RAGAS faithfulness ≥0.75, avg_score ≥0.65 trên 50 câu.
- Attacks ≥18/20 cho CI đề xuất; clean 8/8 allowed; output cases đúng.
- Full guard P95 <500ms chỉ áp dụng sau khi thực sự đo full guard.

Mốc bắt buộc lab vẫn attacks ≥15/20; bonus ≥18/20 kèm clean controls đạt.

### Đề xuất Job CI/CD và Script kiểm tra trả mã Nonzero:
Trong file cấu hình CI (ví dụ `.github/workflows/eval_guard_gate.yml`), job kiểm định chất lượng được thiết lập chạy tự động trên mỗi Pull Request:
```bash
# Lệnh chạy CI gate trong pipeline:
python scripts/run_experiments.py --phase all --live --latency-samples 20
python scripts/check_lab.py --with-tests --ci-mode
```
Script kiểm tra CI (`scripts/ci_gate_check.py`) sẽ đọc trực tiếp các artifacts JSON và trả về exit code 1 (khiến CI build fail và chặn merge PR) nếu vi phạm bất kỳ điều kiện nào sau đây:
- `ragas_50q.json`: faithfulness trung bình < 0.75 hoặc avg_score < 0.65.
- `guard_results.json`: `adversarial.passed` < 18/20 hoặc `clean.false_positive_rate` > 0.0 (bất kỳ câu hợp lệ nào bị chặn đều fail CI).
- `guard_results.json`: bất kỳ ca nào trong `output_results` bị `passed == False`.
- `judge_results.json`: Cohen's kappa < 0.60 hoặc Position Bias Rate > 0.10.

*Lưu ý provenance*: Đây là đề xuất kiến trúc CI/CD hoàn chỉnh dựa trên các chỉ số thực tế của bài lab, workflow trên hiện chưa triển khai tự động trong repo mà là blueprint kiến trúc phục vụ triển khai sản xuất.

## Monitoring và cải tiến

### Giám sát vận hành (Production Monitoring):
1. **Chỉ số theo dõi liên tục (Continuous Metrics)**:
   - Tỷ lệ vi phạm Guardrail Input/Output theo thời gian thực (Jailbreak rate, PII detection rate).
   - Tỷ lệ từ chối sai (False Positive Rate) thu thập từ phản hồi người dùng (nút "Report issue / False refusal").
   - Độ trễ P50, P95, P99 của từng tầng (Presidio, NeMo Input, RAG retrieval, LLM Generation, NeMo Output).
   - Điểm đánh giá định kỳ RAGAS Faithfulness và Context Recall trên mẫu 5% số lượng truy vấn thực tế được ẩn danh hóa.
2. **Ngưỡng cảnh báo (Alert Thresholds)**:
   - Cảnh báo vàng: Tỷ lệ vi phạm guardrail tăng đột biến > 15% trong 1 giờ (dấu hiệu tấn công dò quét hệ thống).
   - Cảnh báo đỏ: Độ trễ P95 input guard vượt quá 1200ms trong 5 phút liên tục, hoặc tỷ lệ khiếu nại false positive > 1%.
3. **Quy trình xử lý sự cố (Incident Response)**:
   - Kỹ sư trực On-call (MLOps/SecOps) tiếp nhận cảnh báo qua Slack/PagerDuty, trích xuất request_id đã được che mờ PII để phân tích log vi phạm.
   - Khi phát hiện prompt injection mới lọt qua lớp bảo vệ, lập tức bổ sung test case vào bộ `adversarial_set` và cập nhật Colang rules/few-shot prompts.

### Tóm tắt số liệu thực nghiệm làm bằng chứng:
- RAGAS mean: 0.7821 / worst distribution: adversarial (mean: 0.7407).
- Cohen’s κ: 1.0 (khớp tuyệt đối giữa Judge và Human annotator trên 10 câu calibration).
- Attacks passed: 19/20 (tỷ lệ thành công 95.0%); clean false_positive_rate: 0.0 (0/8 câu bị chặn nhầm, 100% clean allowed).

### Các đề xuất cải tiến có thể kiểm chứng:
1. **Triển khai Mô hình SLM Guardrail Cục bộ**: Thay thế việc gọi OpenAI API từ xa cho NeMo Input Guard bằng mô hình phân loại cục bộ chạy trên GPU server nội bộ nhằm giảm P95 latency từ 1047ms xuống dưới 100ms.
2. **Mở rộng Từ điển Thực thể Tiếng Việt cho Presidio**: Tích hợp thêm nhận diện mã số thuế doanh nghiệp (MST), số tài khoản ngân hàng nội địa Việt Nam và biển số xe để nâng cao năng lực bảo vệ dữ liệu nhân sự.
3. **Cải thiện Prompt Reranker & Metadata Filtering**: Áp dụng lọc siêu dữ liệu theo phiên bản tài liệu (`valid_from`, `version: 2024`) trước khi ranking để triệt tiêu hoàn toàn lỗi version conflict ở câu hỏi Q39 và Q41 trong bộ RAGAS.
4. **Mở rộng Bộ Calibration Human-in-the-loop**: Gia tăng kích thước tập dữ liệu đối chiếu con người từ 10 câu lên 100 câu đa dạng các phòng ban để kiểm chứng độ ổn định lâu dài của hệ thống LLM Judge.
