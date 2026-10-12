# Phân tích lỗi Phase A

Nhóm: Nhóm cá nhân - Thân Tiến Đạt

Người phụ trách: Thân Tiến Đạt - 2A202603023

Nguồn: `artifacts/ragas_50q.json`; run timestamp: 2026-10-12T09:33:06.552751+00:00.
Generation mode/fallback và điều kiện chạy: 100% LLM mode (gpt-4o-mini-2024-07-18), extractive_fallback = 0 câu, chạy thực nghiệm live có kết nối mạng với OpenAI API key, Qdrant in-memory và HuggingFace cache BAAI/bge-m3.
Models generation/evaluator: Generation: ["gpt-4o-mini-2024-07-18"]; Evaluator: ["BAAI/bge-m3", "gpt-4o-mini"].

## Điểm theo distribution

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 0.9333 | 0.5486 | 0.8000 |
| answer_relevancy | 0.7507 | 0.7194 | 0.6130 |
| context_precision | 0.9083 | 0.9417 | 0.9000 |
| context_recall | 0.8250 | 0.7958 | 0.6500 |
| avg_score | 0.8543 | 0.7514 | 0.7407 |

## Bottom 10 và nguyên nhân

| Rank | ID | Question | Distribution | Avg Score | Worst Metric | Diagnosis | Suggested Fix |
|---|---|---|---|---|---|---|---|
| 1 | 39 | So sánh yêu cầu mật khẩu giữa policy v1.0 và v2.0 về độ dài tối thiểu, thời hạn đổi và MFA. | multi_hop | 0.0000 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 2 | 5 | Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt? | factual | 0.2500 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 3 | 50 | Nhân viên Manager có thể dùng VPN cá nhân (như NordVPN) khi WFH để tăng bảo mật thêm không? | adversarial | 0.3333 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 4 | 33 | Nhân viên Manager có thâm niên 12 năm: tổng phụ cấp hàng tháng và số ngày phép năm theo v2024 là bao nhiêu? | multi_hop | 0.3750 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 5 | 44 | Bao lâu phải đổi mật khẩu một lần? | adversarial | 0.4583 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 6 | 9 | Nam nhân viên được nghỉ bao nhiêu ngày khi vợ sinh con? | factual | 0.5000 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 7 | 30 | So sánh quyền lợi bảo hiểm giữa nhân viên thử việc và nhân viên chính thức. | multi_hop | 0.5625 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 8 | 48 | Nhân viên thử việc có được hưởng bảo hiểm sức khỏe PVI không? | adversarial | 0.6667 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 9 | 38 | Nhân viên đi công tác nước ngoài 4 ngày tại thành phố lớn, ở khách sạn 200 USD/đêm (3 đêm). Công ty thanh toán bao nhiêu tiền khách sạn? | multi_hop | 0.7069 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 10 | 27 | Nhân viên có thâm niên 7 năm theo v2024 được nghỉ bao nhiêu ngày phép? Nếu bị trừ thêm 4 ngày ốm không có giấy y tế thì còn lại bao nhiêu? | multi_hop | 0.7130 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |

### Phân tích các ca lỗi điển hình:
1. **Lỗi Hallucination & Version Conflict (Q39 - Rank 1, Avg Score 0.0000)**:
   - *Câu hỏi*: So sánh yêu cầu mật khẩu giữa policy v1.0 và v2.0.
   - *Nguyên nhân*: Context retrieval lấy được cả văn bản chính sách cũ và mới nhưng LLM tổng hợp bị nhầm lẫn giữa các mốc thời gian, tạo ra câu trả lời không bám sát context (`faithfulness = 0.0`). Cần bổ sung metadata filtering theo version hoặc đưa chỉ thị rõ ràng trong prompt yêu cầu phân tách rành mạch giữa các phiên bản.
2. **Lỗi Retrieval / Prompt Matching (Q5 - Rank 2, Avg Score 0.2500)**:
   - *Câu hỏi*: Mua thiết bị trị giá 55 triệu cần ai phê duyệt?
   - *Context*: Context chứa quy định chung về mua sắm nhưng đoạn quy định cụ thể mức trên 50 triệu (cần Tổng Giám đốc duyệt) bị trôi xuống thứ hạng thấp, dẫn đến việc LLM trả lời "cần phê duyệt Kế toán trưởng" (nhầm sang quy định tạm ứng). Điều này khiến `answer_relevancy` và `context_precision` bị kéo xuống 0.
3. **Lỗi Adversarial Negation (Q50 - Rank 3, Avg Score 0.3333)**:
   - *Câu hỏi*: Nhân viên Manager có thể dùng VPN cá nhân (NordVPN) khi WFH không?
   - *Nguyên nhân*: Đây là câu hỏi bẫy adversarial dạng phủ định. Chính sách quy định rõ cấm dùng VPN cá nhân, chỉ được dùng VPN công ty cấp. Tuy nhiên mô hình LLM bị bẫy bởi câu hỏi mang tính gợi ý khẳng định ("để tăng bảo mật thêm"), dẫn đến việc trả lời thiếu kiên quyết hoặc hallucinate điều kiện cho phép.

## Matrix và nhóm yếu nhất

| Metric | factual | multi_hop | adversarial | Tổng |
|---|---|---|---|---|
| faithfulness | 2 | 13 | 2 | 17 |
| answer_relevancy | 12 | 3 | 2 | 17 |
| context_recall | 5 | 4 | 4 | 13 |
| context_precision | 1 | 0 | 2 | 3 |
| **Tổng số câu** | **20** | **20** | **10** | **50** |

- Nhóm yếu theo mean thấp nhất: `adversarial`; mean: 0.7407.
- Worst metric xuất hiện nhiều nhất: `faithfulness`; số câu: 17 (đồng hạng với `answer_relevancy` 17 câu).

### Giải thích nguyên nhân nhóm adversarial yếu nhất:
- Nhóm `adversarial` đạt điểm trung bình thấp nhất (0.7407) vì tập câu hỏi này được thiết kế có chủ đích với các kỹ thuật bẫy: câu hỏi tiền giả định sai, câu hỏi thăm dò thông tin bị cấm (như VPN cá nhân, mật khẩu mặc định), hoặc câu hỏi dựa trên chính sách đã hết hiệu lực. 
- Mặc dù `context_precision` của nhóm này vẫn đạt 0.9000 nhờ BGE-M3 và BGE-reranker-v2-m3, nhưng `context_recall` chỉ đạt 0.6500 và `answer_relevancy` chỉ đạt 0.6130. Khi gặp câu hỏi bẫy, LLM thường có xu hướng trả lời vòng vo hoặc rơi vào bẫy thay vì từ chối dứt khoát hoặc chỉ ra mâu thuẫn.
- **Đề xuất khắc phục**: 
  - Cập nhật prompt system với quy tắc guardrail nghiêm ngặt: "Nếu câu hỏi chứa giả định trái ngược với chính sách hoặc hỏi về công cụ bị cấm, phải bác bỏ trực tiếp và trích dẫn điều khoản cấm".
  - Chạy lại thực nghiệm bằng lệnh `python scripts/run_experiments.py --phase a --live` và so sánh mean avg_score của nhóm adversarial xem có tăng lên trên 0.80 hay không.

## Bẫy adversarial và giới hạn

- So với nhóm `factual` (mean 0.8543), cả hai nhóm `multi_hop` (mean 0.7514) và `adversarial` (mean 0.7407) đều gặp thử thách lớn hơn rõ rệt. Nhóm factual có độ tin cậy (`faithfulness` đạt 0.9333) vượt trội, trong khi ở multi-hop, việc phải nối ghép từ nhiều văn bản khiến faithfulness sụt giảm mạnh xuống 0.5486.
- Bẫy adversarial điển hình trong bộ dữ liệu tập trung vào 2 dạng chính:
  1. *Version conflict*: Đưa ra câu hỏi liên quan đến chính sách có nhiều phiên bản (v2023 vs v2024 như số ngày phép năm, quy định mật khẩu). Nếu retriever không ưu tiên bản mới nhất, câu trả lời sẽ bị sai lệch hoàn toàn.
  2. *Negation / Prohibition inquiry*: Hỏi về hành vi bị cấm nhưng diễn đạt theo hướng tích cực (như Q50 về dùng NordVPN để bảo mật). Điểm thấp của pipeline phản ánh việc hệ thống RAG thuần túy chưa có lớp guardrail chặn trước.
- **Giới hạn thực nghiệm**:
  - Tập đánh giá gồm 50 câu (20 factual, 20 multi_hop, 10 adversarial), phản ánh xu hướng nhưng kích thước mẫu còn khiêm tốn.
  - Sử dụng LLM generator `gpt-4o-mini-2024-07-18`, mô hình có thể có bias nhất định đối với các hướng dẫn dài.
  - 100% các câu hỏi đều chạy thành công qua LLM generation, không có câu nào phải kích hoạt `extractive_fallback`.
