# Block 1 - Tra cứu chính sách đúng phiên bản

Ngày cập nhật: 2026-10-07
Trạng thái: Triển khai và kiểm tra offline đã có bằng chứng; trace A/B đã đối chiếu cho Stage 02 gốc và bản Submit có `list_files`.

## Mục tiêu và phạm vi

Block 1 yêu cầu cài đặt `list_files` ở Stage 01, đăng ký tool ở Stage 02, tạo skill `refund-policy`, và kiểm tra agent tìm đúng tài liệu chính sách theo ngày mua. Agent cần hỏi lại nếu thiếu ngày mua, ngày yêu cầu hoàn tiền hoặc trạng thái kích hoạt. Bài cũng yêu cầu đổi tên file chính sách, mở hội thoại mới và chạy lại trường hợp A/B để xác nhận agent tìm file theo nội dung thay vì phụ thuộc tên cũ.

## Thay đổi đã triển khai

- Stage 01 Submit và Stage 02 Submit có `list_files`, liệt kê các mục trực tiếp (file và thư mục), sắp xếp ổn định theo tên, trả path tương đối workspace và dùng chung kiểm tra path để chặn đường dẫn thoát workspace.
- Agent export và đăng ký `list_files`; capability trong prompt, UI và README được cập nhật.
- Stage 01/02 có hai file chính sách trong `workspace/data/policies/` và fixtures tương ứng để khôi phục workspace.
- Stage 02 có skill `refund-policy` cùng answer template trong workspace và fixtures. Skill hướng dẫn tìm tài liệu trong `data/policies/`, chọn chính sách theo ngày mua và hỏi lại khi thiếu thông tin bắt buộc.

## Kiểm tra mã offline

| Project | Kết quả đã ghi nhận |
|---|---|
| Stage 01 Submit | 39 passed, 3 skipped |
| Stage 02 Submit | 46 passed, 3 skipped |

Ba test bị skip ở mỗi project là các test symlink trên Windows, do môi trường không có quyền tạo symlink. Các test offline có mock model không chứng minh hành vi của model thật.

## So sánh trace Stage 0, Stage 1 và Stage 2

Các trace Stage 0/1 bên dưới dùng prompt tóm tắt ghi chú tuần, không phải ca tra cứu refund-policy. Chúng chỉ cho thấy thay đổi về khả năng tool và mức độ hoàn tất của các lượt đã ghi.

| Stage | Trace | Quan sát được |
|---|---|---|
| Stage 0 | `stage-00-chat/traces/20261007-205018_76e5cffa_turn01_29c78a99.jsonl`, `20261007-205048_76e5cffa_turn02_7d17c2c4.jsonl` | Không có tool trong request và không có sự kiện tool. Lượt hỏi đọc `data/weekly_notes.md` hoàn tất nhưng không có bằng chứng đọc file từ tool. |
| Stage 1 gốc | `stage-01-files/traces/20261007-210541_81b7bcda_turn01_d51c0dda.jsonl` | Đọc `data/weekly_notes.md` thành công (event 5), ghi `output/summary.md` thành công với `status=created` (event 9), rồi lượt chạy hoàn tất. |
| Stage 1 Submit | `stage-01-files-Submit/traces/20261007-212708_a644e8ce_turn01_f242a561.jsonl`; `20261007-212719_a644e8ce_turn02_a544da8b.jsonl`; `20261007-213314_a644e8ce_turn03_14d33251.jsonl` | Trace đầu kết thúc sau yêu cầu `write_file`, chưa có kết quả tool ghi. Trace sau trả `FILE_NOT_FOUND` cho file không tồn tại. Trace Turn 3 gọi `list_files` qua các cấp thư mục và tìm thấy ghi chú, hai policy, `output/summary.md`; không đọc nội dung policy. |
| Stage 2 gốc | `stage-02-skills/traces/20261007-214320_94d76fd1_turn01_92770f26.jsonl` | Lượt tạo báo cáo tuần đọc skill/template và ghi file thành công; đây không phải ca refund-policy. |
| Stage 2 gốc | `stage-02-skills/traces/20261007-214726_31c58523_turn01_3b413e43.jsonl`; `20261007-214743_31c58523_turn02_afcc0236.jsonl` | Hai lượt refund A/B thuộc conversation `31c585231d3a46edae14a3271a637260`. Request chỉ có `read_file`, không có `list_files`; Turn 1 không gọi tool, Turn 2 hoàn tất nhưng dùng câu trả lời chung, yêu cầu kiểm tra chính sách của nhà cung cấp. Không có bằng chứng đọc skill hay policy. |
| Stage 2 Submit | `stage-02-skills-Submit/traces/20261007-214729_f694ea47_turn01_21979c2e.jsonl`; `20261007-214746_f694ea47_turn02_f9e7a1e5.jsonl` | Bản có `list_files` đọc `refund-policy/SKILL.md`, liệt kê `data/policies`, đọc nội dung hai policy và answer template. Turn 1 chọn policy trước tháng 10; Turn 2 chọn policy từ tháng 10. Kết quả câu trả lời nằm trong snapshot hội thoại của trace (xem chi tiết bên dưới). |

Trace Stage 2 Submit xác nhận lần tra cứu theo nội dung hoạt động được cho A/B: skill được nạp, `list_files` phát hiện tài liệu, và `read_file` đọc policy trước khi trả lời. Hai file policy đều được đọc ở mỗi lượt; việc chọn đúng căn cứ được thể hiện trong câu trả lời cuối. Kết luận này chỉ áp dụng cho tên file ban đầu trong trace; chưa có trace đổi tên file.

## Kết quả theo tình huống A/B

| Stage / tình huống | Kết quả kỳ vọng | Kết quả quan sát | Đánh giá |
|---|---|---|---|
| Stage 02 gốc - A: mua 28/09/2026, yêu cầu 06/10/2026, chưa kích hoạt | Policy trước tháng 10; 8 ngày; quá hạn 7 ngày nên không đủ điều kiện | Turn 1 hoàn tất (event 4), không gọi tool. Không lưu được nội dung câu trả lời của Turn 1 trong trace. | Chưa xác minh; không có bằng chứng tra cứu |
| Stage 02 gốc - B: mua 02/10/2026, yêu cầu 12/10/2026, chưa kích hoạt | Policy từ tháng 10; 10 ngày; đủ điều kiện trong 14 ngày, không phí | Turn 2 (event 2) nói còn tùy chính sách nhà cung cấp và đề nghị kiểm tra điều khoản; không dùng policy trong workspace. Hoàn tất ở event 4. | Chưa đạt; chưa tra cứu/chưa trả lời theo policy |
| Stage 02 Submit - A | Policy trước tháng 10; 8 ngày; không đủ điều kiện | Gọi `list_files("data/policies")` (events 7-9), đọc skill/policy/template thành công (events 11-17), hoàn tất event 20. Câu trả lời trong snapshot: “Chính sách áp dụng: Chính sách hoàn tiền trước tháng 10… 8 ngày… Không đủ điều kiện…”, dẫn `data/policies/policy-before-oct.md`. | Đạt theo trace và nội dung policy |
| Stage 02 Submit - B | Policy từ tháng 10; 10 ngày; đủ điều kiện, không phí | Turn 2 (event 1) là câu hỏi B; đọc skill và cả hai policy thành công (events 3-13), sau đó hoàn tất event 16. Câu trả lời gắn với lượt B trong snapshot: policy từ tháng 10, 10 ngày, đủ điều kiện do trong giới hạn 14 ngày và chưa kích hoạt, không phí; dẫn `data/policies/policy-from-oct.md`. | Đạt theo trace và nội dung policy |

**Ghi chú về nội dung trả lời:** trace Stage 02 Submit ghi `answer_chars` ở `run_completed`, nhưng không có riêng event cuối chứa câu trả lời; nội dung trên được khôi phục từ snapshot `model_request` tiếp theo. Với B, snapshot lượt kế tiếp chứa câu trả lời A trước đó cùng hội thoại; phần trả lời được gắn với prompt B là câu assistant cuối cùng sau các kết quả tool của B. Không có ảnh UI đính kèm.

## Đối chiếu trace chi tiết

| Tình huống | Trace / conversation ID | Bằng chứng chính |
|---|---|---|
| Stage 02 gốc - A | `stage-02-skills/traces/20261007-214726_31c58523_turn01_3b413e43.jsonl`; conversation `31c585231d3a46edae14a3271a637260` | Event 1 user prompt; event 3 model response không có tool call; event 4 run hoàn tất, `answer_chars=483`. Trace không giữ nội dung trả lời cuối. |
| Stage 02 gốc - B | `stage-02-skills/traces/20261007-214743_31c58523_turn02_afcc0236.jsonl`; conversation `31c585231d3a46edae14a3271a637260` | Event 1 user prompt; event 2 snapshot có câu trả lời chung, không viện dẫn policy workspace; event 3 không có tool call; event 4 hoàn tất, `answer_chars=361`. |
| Stage 02 Submit - A | `stage-02-skills-Submit/traces/20261007-214729_f694ea47_turn01_21979c2e.jsonl`; conversation `f694ea47da494d7f8b17fdcf0261c68b` | Events 1-2 prompt/context; events 3-5 nạp skill; events 7-9 liệt kê `data/policies` thấy hai file; events 11-17 đọc hai policy và template; event 18 snapshot giữ câu trả lời A; events 19-20 hoàn tất (`answer_chars=570`). |
| Stage 02 Submit - B | `stage-02-skills-Submit/traces/20261007-214746_f694ea47_turn02_f9e7a1e5.jsonl`; conversation `f694ea47da494d7f8b17fdcf0261c68b` | Event 1 prompt B; events 3-5 nạp skill; events 7-13 đọc hai policy và template; event 14 snapshot ghi nội dung các lượt trong cùng hội thoại, gồm câu trả lời B; events 15-16 kết thúc model/run (`answer_chars=516`). |

## Ca thiếu trạng thái kích hoạt

Đã có trace mới cho câu hỏi “Tôi mua ngày 02/10/2026, muốn hoàn ngày 12/10/2026. Tôi có được hoàn không?” trên cả bản gốc và Submit.

| Stage | Trace / conversation ID | Quan sát | Đánh giá |
|---|---|---|---|
| Stage 02 gốc | `stage-02-skills/traces/20261007-220234_be53f944_turn01_34785994.jsonl`; conversation `be53f944639944f797b7f459dcd91662` | Event 1 có câu hỏi thiếu trạng thái; event 3 trả lời không gọi tool; event 4 hoàn tất (`answer_chars=580`). Trace không lưu nội dung câu trả lời cuối. | Chưa thể xác minh nội dung câu hỏi lại từ trace JSONL. |
| Stage 02 Submit | `stage-02-skills-Submit/traces/20261007-220237_163dc4e7_turn01_f3aed757.jsonl`; conversation `163dc4e73a234e1ea6abb8e2b83bd71c` | Event 1 có câu hỏi thiếu trạng thái; events 3-5 đọc skill; model trả lời không gọi policy tools (event 7) và kết thúc event 8 (`answer_chars=191`). Snapshot chưa có nội dung assistant cuối. | Bằng chứng luồng phù hợp (đọc skill, không tra policy/kết luận), nhưng cần nội dung UI để xác nhận agent hỏi trạng thái kích hoạt. |

Do trace chỉ có số ký tự trả lời cuối, chưa khẳng định chính xác câu hỏi của model nếu không có bản chép/ảnh UI. Theo tiêu chí lab, ca này chỉ được đánh giá đạt khi câu trả lời quan sát được hỏi trạng thái kích hoạt và chưa kết luận.

## Ca còn chờ

| Tình huống | Trạng thái bằng chứng |
|---|---|
| Đổi tên hai file, mở hội thoại mới, chạy lại A và B | Chưa có trace. Các trace hiện tại đọc tên cũ `policy-before-oct.md` và `policy-from-oct.md`; vì vậy chưa chứng minh khả năng tìm theo nội dung sau khi đổi tên. |

## Giới hạn bằng chứng hiện tại

Đã xác minh bằng trace model thật trong môi trường lab rằng bản Stage 02 Submit dùng skill, `list_files` và `read_file` để tra cứu rồi trả lời đúng hai ca A/B với tên file ban đầu. So sánh Stage 02 gốc cho thấy không có `list_files`; Turn 2 đưa câu trả lời chung thay vì tra cứu tài liệu. Ca thiếu trạng thái đã có trace ở cả hai bản, nhưng nội dung câu trả lời cuối không được ghi vào JSONL; cần bản chép/ảnh UI để chấm chắc việc hỏi lại. Chưa xác minh đổi tên file trong hội thoại mới. Kết quả test offline có mock model chỉ xác nhận mã, schema và luồng mock; không thay thế trace model thật.
