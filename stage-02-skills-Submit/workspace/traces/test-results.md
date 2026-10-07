# Kết quả kiểm tra refund-policy

## Trạng thái hiện tại

Các mục bên dưới là ghi chép từ lượt thử trước khi chuẩn hóa dữ liệu sang `data/policies/`. Chúng chưa được chạy lại sau thay đổi này. Các file JSONL được dẫn trong ghi chép không có trong checkout hiện tại, vì vậy các kết quả model ghi bên dưới chưa thể kiểm chứng độc lập. Xem `analysis.md` ở thư mục gốc của Stage 02 Submit để biết trạng thái mới nhất.

## Cấu hình
- Đã đăng ký: read_file, write_file, list_files.
- Skill: skills/refund-policy/SKILL.md.
- Reference: skills/refund-policy/references/answer-template.md.
- Đường dẫn tìm chính sách hiện tại: data/ (theo xác nhận của người dùng).
- Sau khi phát hiện đường dẫn data/policies/ không đúng với vị trí thực tế của các file policy, đã sửa SKILL.md để tìm tài liệu trong data/.
  Trace turn 01 ghi lỗi của bản cũ, chưa phản ánh bản đã sửa.
- Các trace gốc nằm ở ../../traces/ tính từ file ghi chú này.

## 1. Nạp skill và tìm tài liệu

Trace: [20261007-171155_5e78ef08_turn01_905186a8.jsonl](../../traces/20261007-171155_5e78ef08_turn01_905186a8.jsonl)

Bằng chứng:
- Dòng 5: đọc thành công SKILL.md.
- Dòng 10: đọc thành công answer-template.md.
- Dòng 19: list_files("data") tìm thấy hai file policy.
- Dòng 24–25: đọc thành công cả hai chính sách.

Kết quả: xác nhận agent nạp skill, reference và đọc tài liệu.

## 2. Trường hợp A — mua trước ngày đổi chính sách

Đầu vào:
> Tôi mua ngày 28/09/2026, yêu cầu hoàn ngày 06/10/2026,
> chưa kích hoạt. Tôi có được hoàn không?

Kỳ vọng:
- Chính sách cũ.
- 8 ngày.
- Không đủ điều kiện vì vượt thời hạn 7 ngày.
- Dẫn tài liệu chứa chính sách cũ.

Trace: [20261007-172311_5e78ef08_turn07_59f84c34.jsonl](../../traces/20261007-172311_5e78ef08_turn07_59f84c34.jsonl)
- Dòng 1: chứa đúng đầu vào.
- Lượt này không gọi tool, dùng tài liệu từ lịch sử.
- Trace không chứa nội dung câu trả lời cuối.

Đánh giá: chưa đủ bằng chứng để xác nhận kết quả.
Cần lưu câu trả lời trên giao diện hoặc kiểm tra lại.

## 3. Trường hợp B — mua từ ngày đổi chính sách

Đầu vào được cung cấp qua hai lượt:
- Turn 03: mua 02/10/2026, yêu cầu hoàn 12/10/2026.
- Turn 04: bổ sung sản phẩm chưa kích hoạt.

Bằng chứng kết quả:
Trace: [20261007-172000_5e78ef08_turn05_f9836532.jsonl](../../traces/20261007-172000_5e78ef08_turn05_f9836532.jsonl)
- Dòng 2, snapshot.messages: chứa câu trả lời của turn 04.
- Chọn chính sách mới.
- Tính 10 ngày.
- Kết luận đủ điều kiện, không thu phí.
- Dẫn data/policy-from-oct.md.md.

Đánh giá: đạt kết quả mong đợi.
Chưa chạy nguyên câu hỏi B trong cuộc trò chuyện mới.

## 4. Thiếu trạng thái kích hoạt

Đầu vào:
> Tôi mua ngày 02/10/2026, muốn hoàn ngày 12/10/2026.

Bằng chứng:
Trace: [20261007-171626_5e78ef08_turn04_9d5a762a.jsonl](../../traces/20261007-171626_5e78ef08_turn04_9d5a762a.jsonl)
- Dòng 2, snapshot.messages: chứa câu trả lời của turn 03.
- Agent hỏi: “Sản phẩm của bạn đã được kích hoạt hay chưa?”
- Chưa kết luận đủ/không đủ điều kiện.

Đánh giá: đạt.

## 5. Đổi tên hai file policy

Đã hoán đổi tên hai file, giữ nguyên nội dung.

Bằng chứng:
Trace: [20261007-172000_5e78ef08_turn05_f9836532.jsonl](../../traces/20261007-172000_5e78ef08_turn05_f9836532.jsonl)
- Dòng 5: policy-from-oct.md.md chứa chính sách cũ.
- Dòng 9: policy-before-oct.md.md chứa chính sách mới.
- Agent đọc nội dung và nhận ra chính sách mới nằm trong file
  có tên policy-before-oct.md.md. Câu trả lời được lưu trong
  snapshot.messages, dòng 2 của trace turn 06:
  [20261007-172230_5e78ef08_turn06_0a956544.jsonl](../../traces/20261007-172230_5e78ef08_turn06_0a956544.jsonl).

Đánh giá: xác nhận agent phân biệt chính sách theo nội dung.
Chưa đủ bằng chứng cho bài thử đổi tên:
- Chưa chạy lại cả A và B sau đổi tên.
- Chưa mở cuộc trò chuyện mới; vẫn cùng conversation_id.

Sau kiểm tra, đã khôi phục tên file như ban đầu.

## Việc cần kiểm tra bổ sung
- Tải lại catalog sau khi sửa skill sang data/.
- Chạy A và B trong cuộc trò chuyện mới, lưu câu trả lời.
- Tráo tên policy rồi chạy lại A và B trong cuộc trò chuyện mới.
- Ghi tên trace, conversation_id và bằng chứng cho từng lần chạy.

## Lưu ý về bằng chứng

Các lượt refund đã đối chiếu cùng conversation_id:
5e78ef081fbf490590f9a2262e4f165d.

Trace hiện tại không lưu nội dung câu trả lời cuối trực tiếp trong
model_response hoặc run_completed. Câu trả lời có thể xuất hiện trong
snapshot.messages của lượt tiếp theo, nếu có.

Khi kiểm tra bổ sung, lưu ảnh hoặc bản sao câu trả lời trên giao diện
cùng trace để chứng minh đầy đủ kết quả.
