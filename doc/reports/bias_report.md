# Đánh giá LLM Judge Phase B

Nhóm: Nhóm cá nhân - Thân Tiến Đạt

Người phụ trách: Thân Tiến Đạt - 2A202603023

Nguồn: `artifacts/judge_results.json`; run timestamp: 2026-10-12T09:33:58.342184+00:00.
Models generation/judge: Generation: ["gpt-4o-mini-2024-07-18"]; Judge: ["gpt-4o-mini"].

## Năm cặp A/B và hai lượt

Nguồn cố định: A từ artifacts/answers_50q.json:answer; B từ fixtures/human_labels_10q.json:model_answer; reference từ fixtures/test_set_50q.json:ground_truth.
Ids: 1, 5, 12, 21, 41. Không đổi nguồn sau khi xem kết quả judge.

| ID | Candidate A | Candidate B | Winner Pass 1 | Winner Pass 2 | Final Winner | Consistent | Scores Pass 1 (A/B) | Scores Pass 2 (A/B) | Lý do chính |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ 3 ngày làm việc khi kết hôn. | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | B | B | B | True | 0.7 / 1.0 | 0.8 / 1.0 | Candidate B nêu rõ tính chất có lương của ngày nghỉ kết hôn. |
| 5 | Cần thêm phê duyệt Kế toán trưởng. | Cần Giám đốc phòng ban phê duyệt. | A | A | A | True | 0.7 / 0.3 | 0.5 / 0.2 | Candidate A đề cập thêm phê duyệt Kế toán trưởng sát quy trình hơn B. |
| 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là **1 tháng lương**. | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | A | A | A | True | 1.0 / 0.8 | 1.0 / 0.8 | Candidate A nêu rõ điều kiện thâm niên từ 6 tháng trở lên. |
| 21 | Nhân viên có 9 năm thâm niên được 18 ngày phép (15 + 3) theo chính sách nghỉ phép năm 2024. Số ngày phép năm có lương là 15 ngày. | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | B | B | B | True | 0.7 / 0.9 | 0.7 / 1.0 | Candidate B trả lời trọn vẹn cả vế ngày phép và dải lương Senior. |
| 41 | Nhân viên được nghỉ 12 ngày phép năm trong chính sách năm 2023 và 15 ngày phép năm trong chính sách năm 2024. | Nhân viên được nghỉ 12 ngày phép năm. | A | A | A | True | 1.0 / 0.4 | 0.9 / 0.3 | Candidate A chỉ rõ thay đổi giữa 2 phiên bản và ưu tiên số liệu 2024. |

- Position bias count/rate: 0 / 0.0 (tỷ lệ bias vị trí là 0.0%, 5/5 cặp đều nhất quán giữa 2 lượt swap vị trí).
- Verbosity win rate: 0.8 (80.0%); decisive cases: 5 (trong đó 4 ca thắng là câu trả lời dài hơn, 1 ca thắng là câu trả lời ngắn hơn).

### Phân tích chi tiết về tính nhất quán và Verbosity:
- **Tính nhất quán vị trí (Position Consistency)**: Cả 5 cặp đánh giá đều đạt kết quả nhất quán tuyệt đối (`winner_pass1 == winner_pass2`). Kỹ thuật Two-pass Swap Evaluation đã chứng minh tính hiệu quả cao: khi đảo thứ tự trình bày giữa A và B trong prompt, LLM Judge (gpt-4o-mini) vẫn bảo toàn quyết định đánh giá dựa trên tiêu chí nội dung chứ không thiên vị vị trí xuất hiện trước hay sau.
- **Hiện tượng Verbosity Bias (Thiên vị độ dài)**:
  - Thống kê chi tiết: 3 ca Candidate A thắng và A dài hơn (Q5, Q12, Q41); 1 ca Candidate B thắng và B dài hơn (Q1); 1 ca Candidate B thắng dù B ngắn hơn A (Q21).
  - *Đánh giá bản chất*: Tỷ lệ 0.8 thoạt nhìn là dấu hiệu của verbosity bias, nhưng khi phân tích nội dung thực tế: các câu dài hơn thắng là do chúng chứa đựng nhiều thông tin chính xác và đầy đủ hơn (như phân biệt phiên bản 2023/2024 ở Q41, nêu rõ điều kiện 6 tháng ở Q12, hay chi tiết nghỉ có lương ở Q1). Ở Q21, Candidate B dù ngắn hơn nhưng trả lời đủ cả 2 yêu cầu của câu hỏi nên vẫn giành chiến thắng. Do đó, hiện tượng này phản ánh sự vượt trội về độ đầy đủ (completeness) của thông tin hơn là việc thiên vị số lượng từ ngữ thuần túy.

## Calibration đúng model_answer

Helper đánh giá đúng question/model_answer đang có trong fixtures/human_labels_10q.json; không lấy answer mới của pipeline. Prompt chỉ nhận question, answer, reference. Không gửi human_label, human_note hoặc phân bố nhãn human vào prompt.

| QID | Model Answer Gốc | Nhãn Judge | Reasoning | Human Label | Agree |
|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | 1 | Câu trả lời chính xác và đầy đủ theo thông tin trong tài liệu tham khảo, không thiếu ý thiết yếu. | 1 | True |
| 5 | Cần Giám đốc phòng ban phê duyệt. | 0 | Câu trả lời sai vì cần Tổng Giám đốc phê duyệt chứ không phải Giám đốc phòng ban. | 0 | True |
| 12 | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | 1 | Câu trả lời chính xác theo thông tin trong reference, nêu rõ điều kiện và mức thưởng cho nhân viên chính thức có từ 6 tháng trở lên. | 1 | True |
| 21 | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | 1 | Câu trả lời chính xác theo reference, nêu rõ số ngày phép và khoảng lương cho nhân viên Senior với 9 năm thâm niên. | 1 | True |
| 23 | Hoàn trả 25 triệu vì chưa đủ 1 năm cam kết. | 1 | Câu trả lời chính xác theo reference, nêu rõ nhân viên phải hoàn trả 25 triệu vì chưa đủ 1 năm cam kết. | 1 | True |
| 29 | Trưởng phòng phê duyệt. Phạt 2% tháng trên 8 triệu. | 0 | Câu trả lời thiếu thông tin về việc cần có sự phê duyệt của Kế toán trưởng bên cạnh Trưởng phòng. Ngoài ra, phí phạt được tính cho quá hạn 15 ngày, không phải là 2% trên tổng số tiền tạm ứng. | 0 | True |
| 33 | Phép: 19 ngày. Phụ cấp: 1.500.000 VNĐ/tháng. | 1 | Câu trả lời cung cấp đúng số ngày phép và tổng phụ cấp hàng tháng theo thông tin trong reference. | 1 | True |
| 41 | Nhân viên được nghỉ 12 ngày phép năm. | 0 | Câu trả lời sai vì nó dựa trên chính sách cũ (v2023) thay vì chính sách hiện hành (v2024), trong đó quy định nhân viên được nghỉ 15 ngày phép năm. | 0 | True |
| 46 | Nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần. | 1 | Câu trả lời chính xác theo reference, nêu rõ rằng nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần, đồng thời không bỏ qua điều kiện phê duyệt từ trưởng phòng. | 1 | True |
| 50 | Được, miễn là đảm bảo kết nối an toàn. | 0 | Câu trả lời sai vì cho phép sử dụng VPN cá nhân trong khi chính sách rõ ràng cấm điều này. | 0 | True |

- Cohen’s κ: 1.0 (khớp tuyệt đối giữa Judge và Human Annotator).
- Observed agreement: 10/10 = 1.0000 (100.0%). Chance agreement: (6×6 + 4×4) / 100 = 0.5200 (52.0%).
- Công thức: κ = (P_o - P_e) / (1 - P_e) = (1.0 - 0.52) / (1.0 - 0.52) = 1.0.
- Diễn giải: Giá trị Cohen's kappa đạt 1.0 (vượt xa ngưỡng yêu cầu 0.60), chứng minh LLM Judge có khả năng phân biệt câu trả lời đạt và không đạt hoàn toàn tương đồng với chuyên gia con người trên tập mẫu thử nghiệm.

## Giới hạn và cách dùng judge

- **Mô hình và Prompt**: Sử dụng mô hình `gpt-4o-mini` với few-shot calibration prompt có cấu trúc chặt chẽ, yêu cầu định dạng JSON kèm lý do phân tích (chain-of-thought ngắn) trước khi đưa ra nhãn hoặc lựa chọn winner.
- **Giới hạn số lượng mẫu**:
  - Kích thước tập calibration gồm 10 câu hỏi và pairwise gồm 5 cặp câu hỏi. Mặc dù đạt κ = 1.0, nhưng kích thước mẫu còn nhỏ nên chưa thể đại diện hoàn toàn cho mọi tình huống ngôn ngữ phức tạp trong thực tế sản xuất.
  - LLM Judge cần luôn được coi là công cụ đánh giá tự động bổ trợ (secondary evaluation tool), không thể thay thế hoàn toàn việc kiểm định chất lượng định kỳ từ con người (human audit).
- **Khuyến nghị áp dụng trong Production**:
  - Sử dụng cơ chế Two-pass Swap cho mọi đánh giá A/B testing tự động để triệt tiêu Position Bias.
  - Giám sát độ dài câu trả lời để phát hiện sớm các trường hợp Verbosity bias khi đánh giá các model sinh câu dài dòng nhưng thiếu ý.
  - Thiết lập ngưỡng Confidence / Agreement: các câu có điểm chấm chênh lệch giữa 2 lượt hoặc câu có lý do không rõ ràng cần được đưa vào hàng đợi cho con người duyệt lại (Human-in-the-loop).
