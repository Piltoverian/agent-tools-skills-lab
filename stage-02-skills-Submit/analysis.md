# Block 1 - Báo cáo tổng hợp

Ngày cập nhật: 2026-10-07
Phạm vi: hướng dẫn Block 1, Stage 01 Submit và Stage 02 Submit. Đã sửa code, fixtures, test và metadata để khớp bài; test offline đã chạy. Chưa chạy model thật.

## Kết quả

**Đạt phần triển khai và kiểm tra offline. Chưa đủ hồ sơ để xác nhận hoàn tất toàn bộ Block 1** vì chưa có trace mới từ các tình huống model thật.

- `list_files` ở hai stage liệt kê các mục trực tiếp, gồm file và thư mục, theo thứ tự tên; trả đường dẫn workspace tương đối và loại `file`/`directory`. Kiểm tra path dùng chung `_resolve` để chặn traversal, absolute path và symlink thoát workspace.
- Cả hai agent export và đăng ký `list_files`; capability trong prompt, UI và README được cập nhật.
- Test cho cả hai stage bao phủ thư mục/file, thứ tự, thư mục rỗng, path không tồn tại, path là file, traversal, absolute path, symlink escape và tool schema/agent flow.
- Policy có tên chuẩn `policy-before-oct.md`, `policy-from-oct.md` tại `workspace/data/policies/` của cả hai stage và trong `fixtures/data/policies/` để reset vẫn khôi phục dữ liệu.
- Stage 02 có `refund-policy` và answer template trong workspace cùng fixtures. Skill tra cứu `data/policies/`, chọn chính sách theo ngày mua và hỏi lại khi thiếu thông tin.

## Kiểm tra offline

Chạy từ thư mục riêng của từng project bằng `uv run --locked pytest`:

| Project | Kết quả |
|---|---|
| Stage 01 Submit | 39 passed, 3 skipped |
| Stage 02 Submit | 46 passed, 3 skipped |

Ba test bị skip ở mỗi project là các test symlink: Windows trả `WinError 1314` do phiên chạy không có quyền tạo symlink. Các test khác chạy qua, gồm AppTest với mock model, đăng ký/schema tool, gọi `list_files` rồi `read_file`, catalog skill và project structure.

## Còn thiếu trước khi xác nhận hoàn tất

Hướng dẫn yêu cầu kiểm tra A, B, trường hợp thiếu trạng thái kích hoạt, và sau khi đổi tên file chạy lại cả A/B trong cuộc trò chuyện mới; cần nộp trace và câu trả lời tương ứng (`lab-guide/exercise-block-1.html:835-836, 868-896`). Các file JSONL được nhắc trong ghi chép cũ không có trong checkout. Ghi chú lịch sử ở `workspace/traces/test-results.md` đã được đánh dấu là chưa chạy lại sau khi chuẩn hóa dữ liệu.

Trong môi trường hiện tại không có `.env`, `OPENAI_API_KEY` hoặc `MODEL_NAME`; vì vậy chưa thể chạy model thật để tạo các trace được yêu cầu. Không tạo trace mô phỏng hoặc ghi nhận kết quả model chưa quan sát.

**Trạng thái cuối:** code, dữ liệu, fixtures và test offline đã được sửa; live A/B, thiếu thông tin, đổi tên trong hội thoại mới và trace tương ứng vẫn **chưa xác minh**.
