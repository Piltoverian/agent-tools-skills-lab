---
name: csv-quality
description: Kiểm tra chất lượng CSV danh sách công việc (task_id, owner, hours), tính tổng giờ hợp lệ theo owner, xác định người quá tải theo ngưỡng do người dùng cung cấp và ghi báo cáo Markdown. Dùng khi người dùng yêu cầu kiểm tra hoặc đánh giá CSV công việc.
---

# CSV quality

Phân tích CSV bằng script có sẵn, không tự đếm bằng mắt.

## Chạy script

Dùng tool `bash` (cwd là workspace). Ngưỡng là bắt buộc, hữu hạn và không âm:

```
python skills/csv-quality/scripts/check_csv.py --input <đường-dẫn-CSV> --max-hours <ngưỡng>
```

Ví dụ: `python skills/csv-quality/scripts/check_csv.py --input data/workload.csv --max-hours 8`.

Nếu yêu cầu quá tải không nêu ngưỡng, hỏi người dùng trước khi kết luận. Không lấy ngưỡng từ cuộc trò chuyện cũ. Không cần đọc source script để chạy; chỉ đọc khi cần hiểu hành vi chưa mô tả ở đây.

## Quy tắc tính

- Bỏ khoảng trắng đầu/cuối của ID, owner và hours; phân biệt owner theo tên và hoa/thường.
- Với mỗi ID không rỗng, lần xuất hiện đầu tiên giữ ID kể cả khi dòng đó có dữ liệu không hợp lệ. Mọi lần sau bị loại là `duplicate_id`.
- Chỉ cộng dòng có đúng số trường, ID và owner không rỗng, ID chưa xuất hiện trước đó, và hours là số hữu hạn không âm. `0` hợp lệ.
- Chỉ cộng các dòng hợp lệ. Bỏ qua dòng trống như script hiện có.
- Quá tải khi tổng giờ lớn hơn ngưỡng; bằng ngưỡng không phải quá tải.

## Đọc kết quả

- Exit code `0`: phân tích thành công, kể cả khi có dòng bị loại hoặc owner quá tải. Stdout là JSON gồm `max_hours`, `hours_by_owner`, `overloaded_owners`, `excluded_rows` và các trường kiểm tra chất lượng hiện có (`row_count`, `missing_owner_count`, `invalid_hours_count`, `duplicate_id_count`, `duplicate_ids`, `issues`).
- `hours_by_owner` chỉ có owner có ít nhất một dòng được cộng. `overloaded_owners` sắp xếp theo tên owner.
- `excluded_rows` sắp xếp theo line, mỗi dòng xuất hiện một lần; `task_id` rỗng là `null`. Reasons luôn theo thứ tự: `wrong_field_count`, `missing_task_id`, `duplicate_id`, `missing_owner`, `invalid_hours`.
- Exit code khác `0`: lỗi thực thi như thiếu tham số, ngưỡng không hợp lệ, file không đọc được, thiếu cột hoặc lỗi parse. Đọc stderr, báo lỗi; không bịa thống kê hay ghi báo cáo như thể phân tích thành công.
- Phân biệt lỗi chất lượng (nằm trong JSON `issues` và `excluded_rows`) với lỗi thực thi (exit code khác `0`). Các thống kê chất lượng cũ tiếp tục xét mọi dòng dữ liệu, kể cả dòng không được cộng giờ.

## Viết báo cáo

1. Đọc `references/report-template.md`.
2. Lấy ngưỡng, tổng giờ, danh sách quá tải, dòng bị loại và thống kê chất lượng từ JSON. Issue ghi line number (header là line 1), cột và mô tả.
3. Không sửa CSV khi người dùng chỉ yêu cầu kiểm tra; có thể đề xuất cách sửa.
4. Ghi báo cáo bằng `write_file` vào đường dẫn người dùng yêu cầu (mặc định `output/csv-quality.md`), rồi trả đường dẫn và tóm tắt ngắn.
