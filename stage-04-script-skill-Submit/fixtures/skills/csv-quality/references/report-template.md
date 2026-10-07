# Báo cáo kiểm tra CSV: `{đường dẫn CSV}`

Công cụ: `skills/csv-quality/scripts/check_csv.py` | Exit code: {exit_code}

## Tổng quan

- Ngưỡng tối đa: {max_hours} giờ
- Tổng giờ theo owner: {hours_by_owner}
- Owner quá tải (tổng lớn hơn ngưỡng): {overloaded_owners}
- Số dòng dữ liệu (không tính header): {row_count}
- Dòng thiếu owner: {missing_owner_count}
- Dòng hours không hợp lệ: {invalid_hours_count}
- Số task_id lặp (distinct): {duplicate_id_count} ({duplicate_ids})

## Dòng bị loại khỏi phép cộng giờ

Mỗi dòng liệt kê một lần; reasons theo thứ tự ổn định.

| Line | task_id | Reasons |
|---|---|---|
| {line} | {task_id} | {reasons} |

## Chi tiết vấn đề chất lượng

| Line | Cột | Loại | task_id | Mô tả |
|---|---|---|---|---|
| {line} | {column} | {type} | {task_id} | {message} |

## Đánh giá

- Owner quá tải: {nêu owner và tổng giờ; bằng ngưỡng không quá tải}.
- Lý do các dòng bị loại và ảnh hưởng đến phép cộng: {tóm tắt từ JSON}.

## Khuyến nghị

- {Cách sửa đề xuất theo nhóm lỗi; không tự sửa file nguồn}
