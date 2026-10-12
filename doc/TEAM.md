# Nhóm thực hiện K4 Track 3B

Tên nhóm: ThanTienDat-Eval-Guard

Repo chung: https://github.com/Datbadboiz11/K4-Track-3B-Day24-Eval-Guard

Trưởng nhóm/người nộp: Thân Tiến Đạt - MSSV: 2A202603023

Ngày lab/deadline áp dụng: 23:59 (13/10/2026) — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: d98f9b4

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; mẫu ba rows không ép nhóm phải có ba người.

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Thân Tiến Đạt | 2A202603023 | Trưởng nhóm, phụ trách toàn bộ pipeline, Phase A, B, C và Blueprint | Commit d98f9b4, artifacts và reports |

Người tích hợp/chạy batch/nộp link: Thân Tiến Đạt, tích hợp toàn bộ pipeline và chạy thực nghiệm, nộp repo.

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: Windows 11 x64, Python 3.12.4, MSVC Build Tools.
Virtualenv/dependencies/imports: Đã cài đặt dependencies từ requirements.txt, offline tests đạt 86/86 PASS.
Models/Qdrant/50 answers đã chuẩn bị: Model BAAI/bge-m3 và bge-reranker-v2-m3 đã cache sẵn; Qdrant in-memory fallback; 50 answers tạo từ setup_answers.py.
Nếu cài full NeMo chưa thành công, ghi lỗi thực tế; không dùng unit test pass hoặc pip resolve để khẳng định runtime đã chạy.

## INDIVIDUAL

### ThanTienDat-2A202603023

- Phần được giao và phần thực tế đã làm: Làm việc độc lập toàn bộ bài lab. Xây dựng và tích hợp 5 module RAG từ Day 18 (m1_chunking, m2_search, m3_rerank, m4_eval, m5_enrichment). Hoàn thành 12 Tasks trong 3 Phase: Phase A RAGAS evaluation và failure clustering, Phase B LLM-as-a-Judge với swap-and-average và Cohen kappa, Phase C Presidio PII và NeMo Guardrails kèm đo P95 latency.
- Bằng chứng: Mã nguồn trong src/, toàn bộ 86 unit tests offline PASS, các file kết quả artifacts/*.json và 3 báo cáo failure_clusters.md, bias_report.md, blueprint.md.
- Cách kiểm tra: Chạy unit tests offline với pytest không dùng mạng; chạy runner thực nghiệm live với OpenAI API và đối chiếu bằng scripts/check_lab.py đạt pass toàn bộ.
- Điều đã học: Hiểu rõ cơ chế đánh giá RAGAS (faithfulness, relevancy, precision, recall), cách khắc phục position bias của LLM judge bằng swap-and-average, ý nghĩa thống kê của Cohen kappa, và thiết kế luồng phòng thủ đa tầng input/output rails bảo vệ hệ thống AI.
- Giới hạn/cách xử lý: API key cũ từ Day 18 bị hết hạn, đã xử lý bằng cách cập nhật API key mới; NeMo Guardrails chạy input layer sequential đúng phạm vi đo đạc.
- Nguồn/AI hỗ trợ nếu có: Sử dụng Antigravity coding assistant hỗ trợ rà soát cấu trúc code và hướng dẫn thực thi; tự kiểm chứng toàn bộ logic bằng pytest và check_lab.py.

## Cập nhật sau khi nộp

Bản nộp chính thức đúng hạn, không có cập nhật chỉnh sửa sau deadline.
