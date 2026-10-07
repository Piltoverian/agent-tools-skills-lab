# Báo cáo tổng hợp

Ngày cập nhật: 2026-10-07
Phạm vi hiện có: Block 1 của lab Agentic AI Engineering. Báo cáo Block 1 chi tiết nằm tại [block-1.md](block-1.md).

## Tình trạng

| Hạng mục | Trạng thái | Bằng chứng / ghi chú |
|---|---|---|
| Block 1 - tra cứu chính sách đúng phiên bản | Đã triển khai, kiểm tra offline và đối chiếu trace A/B trên Stage 02 gốc/Submit; ca thiếu thông tin đã có trace nhưng còn thiếu nội dung UI để chấm; ca đổi tên file đang chờ | Stage 01/02 Submit và [báo cáo Block 1](block-1.md) |
| Các block khác | Chưa tổng hợp trong báo cáo này | Sẽ bổ sung khi có phạm vi và bằng chứng tương ứng |

## Tóm tắt Block 1

Block 1 mở rộng agent qua hai stage. Stage 01 bổ sung tool `list_files` để tìm file trong workspace. Stage 02 đăng ký tool này và tạo skill `refund-policy` để tìm, đọc chính sách và chọn phiên bản theo ngày mua. Chính sách mẫu nằm trong `workspace/data/policies/`; skill tham chiếu đường dẫn tương đối workspace là `data/policies/`.

Kiểm tra tự động offline đã được ghi nhận trước đó: Stage 01 Submit có 39 test đạt và 3 test symlink bị bỏ qua; Stage 02 Submit có 46 test đạt và 3 test symlink bị bỏ qua do môi trường Windows không cấp quyền tạo symlink. Trace Stage 02 gốc cho A/B không có `list_files` và không chứng minh được việc đọc policy; ở Submit, trace xác nhận agent nạp skill, liệt kê thư mục policy, đọc hai file và trả lời A/B phù hợp với tài liệu. A: 8 ngày, quá hạn chính sách 7 ngày, không đủ điều kiện. B: 10 ngày, nằm trong giới hạn 14 ngày, đủ điều kiện và không phí. Chi tiết event, conversation ID và giới hạn cách lưu câu trả lời nằm trong [báo cáo Block 1](block-1.md).

## Bằng chứng còn chờ

Đã có kết luận có giới hạn cho A/B trên Stage 02 Submit và so sánh với bản gốc. Ca thiếu trạng thái đã có trace trên cả Stage 02 gốc và Submit; JSONL không lưu nội dung câu trả lời cuối nên cần bản chép hoặc ảnh UI để xác nhận agent đã hỏi trạng thái kích hoạt và chưa kết luận. Còn chờ trace đổi tên file rồi chạy lại A/B trong hội thoại mới.

So sánh trace hiện có: Stage 0 không có tool; Stage 1 gốc đọc file rồi ghi `output/summary.md` thành công; Stage 1 Submit có trace gọi `list_files` qua nhiều cấp thư mục; Stage 2 gốc không tra cứu policy ở A/B; Stage 2 Submit tra cứu và trả lời đúng A/B theo policy gốc. Đã có trace ca thiếu trạng thái cho cả hai bản nhưng chưa có nội dung UI để xác nhận câu hỏi lại. Chưa có bằng chứng đổi tên file. Chi tiết và tên trace nằm trong [báo cáo Block 1](block-1.md). Không đưa API key, `.env` hoặc thông tin bí mật vào báo cáo.

## Cập nhật báo cáo

Khi có thêm trace, ghi tên file, conversation ID, lượt/event bằng chứng, nội dung câu trả lời, đối chiếu kỳ vọng và trạng thái cho từng ca trong báo cáo Block 1 rồi cập nhật bảng tình trạng này.
