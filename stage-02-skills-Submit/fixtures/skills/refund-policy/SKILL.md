---
name: refund-policy
description: >-
  Tra cứu chính sách hoàn tiền, xác định điều kiện hoàn và phí áp dụng
  theo ngày mua, ngày yêu cầu hoàn và trạng thái kích hoạt. Sử dụng khi
  người dùng hỏi có được hoàn tiền không, thời hạn hoàn, phí hoàn hoặc
  chính sách nào áp dụng cho giao dịch của họ.
---

# Kiểm tra chính sách hoàn tiền

## Thông tin cần có

Trước khi kết luận, cần xác định:
- Ngày mua.
- Ngày yêu cầu hoàn tiền.
- Trạng thái kích hoạt sản phẩm hoặc dịch vụ.

Nếu thiếu thông tin, hỏi lại những mục còn thiếu trong một câu hỏi.
Không tự suy đoán ngày yêu cầu là hôm nay hoặc trạng thái chưa kích hoạt.
Nếu ngày nhập mơ hồ, hỏi lại để xác nhận.

## Tra cứu tài liệu

Dùng `list_files` để tìm tài liệu chính sách trong `data/policies/`.
Dùng `read_file` để đọc các tài liệu liên quan.

Đường dẫn truyền vào tool phải tương đối workspace.
Không sử dụng Bash, terminal hoặc script.

Đọc nội dung để xác định:
- Phạm vi áp dụng: sản phẩm, dịch vụ hoặc loại giao dịch.
- Khoảng thời gian hiệu lực.
- Thời hạn yêu cầu hoàn.
- Điều kiện liên quan đến kích hoạt.
- Phí và cách tính phí.

Không chọn chính sách chỉ dựa trên tên file hoặc vì đó là bản mới nhất.
Nếu thiếu thông tin về sản phẩm để xác định phạm vi áp dụng, hỏi lại.

## Chọn và đánh giá chính sách

Chọn chính sách có phạm vi phù hợp và có hiệu lực vào **ngày mua**.
Ngày yêu cầu hoàn dùng để kiểm tra thời hạn, không dùng thay ngày mua
để chọn phiên bản chính sách.

Tính số ngày từ ngày mua đến ngày yêu cầu hoàn.
Áp dụng cách tính ngày và quy tắc bao gồm ngày đầu/cuối nếu tài liệu
quy định. Nếu không quy định, dùng chênh lệch ngày lịch và nêu cách tính.
Nếu ngày yêu cầu trước ngày mua, hỏi lại để xác nhận thay vì kết luận.

Đối chiếu thời hạn, trạng thái kích hoạt và các điều kiện khác trong
chính sách. Không bỏ qua điều kiện chỉ vì giao dịch còn trong thời hạn.

Nếu đủ điều kiện, xác định phí theo tài liệu. Chỉ tính số tiền phí khi
đã có đủ dữ liệu, chẳng hạn giá mua nếu phí tính theo tỷ lệ phần trăm.
Không mặc định miễn phí khi tài liệu không nêu phí.

Nếu không tìm thấy chính sách phù hợp, nhiều chính sách mâu thuẫn,
hoặc quy tắc tại ranh giới hiệu lực chưa rõ, nêu phần chưa xác định
và hỏi bổ sung khi cần. Không tự chọn một bản để kết luận.

## Trả lời

Trước khi soạn câu trả lời, đọc
[reference về định dạng trả lời](references/answer-template.md)
bằng `read_file`, với đường dẫn:
`skills/refund-policy/references/answer-template.md`.

Trả lời theo reference, dẫn đường dẫn tài liệu làm căn cứ.
Chỉ đánh giá điều kiện; không khẳng định đã thực hiện hoàn tiền.
