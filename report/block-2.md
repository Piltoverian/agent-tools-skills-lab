# Block 2 - Tính tổng giờ và phát hiện quá tải

Ngày cập nhật: 2026-10-07
Trạng thái: Stage 03 Submit và Stage 04 Submit có trace ứng dụng Bash sau sửa **PASS**. Trace Stage 04 Submit xác nhận CSV chính, CSV biên, hỏi ngưỡng và file không tồn tại đều được xử lý đúng. Trace Stage 04 gốc trước sửa Bash **FAIL** với RPC `Bash/Service/0x80007072c`.

## Phạm vi

Block 2 dùng Python/Bash để tính giờ theo owner ở Stage 03, sau đó mở rộng skill `csv-quality` ở Stage 04: thêm ngưỡng `--max-hours`, tổng giờ hợp lệ, owner quá tải và các dòng bị loại. Mỗi `task_id` không rỗng chỉ được giữ ở lần xuất hiện đầu tiên, kể cả khi dòng đó có giờ không hợp lệ. Cần giữ thống kê chất lượng cũ, đồng bộ skill vào fixtures và kiểm tra trường hợp biên.

Bài tập được thực hiện trong `stage-03-bash-Submit/` và `stage-04-script-skill-Submit/`. Lỗi Bash Windows được sửa trong Submit; Stage 03 cũng được sửa ở bản gốc theo yêu cầu trước đó. Stage 04 gốc chưa sửa.

## Stage 03 - CSV và Bash runtime

Dữ liệu: `stage-03-bash-Submit/workspace/data/workload.csv`.

| Dòng | Kết quả |
|---|---|
| 2: T01, Lan, 4 | Cộng 4 giờ cho Lan |
| 3: T02, Lan, 5 | Cộng 5 giờ cho Lan |
| 4: T03, Minh, 3 | Cộng 3 giờ cho Minh |
| 5: T04, Minh, abc | Bỏ do giờ không hợp lệ |
| 6: T02, Lan, 5 | Trace Stage 03 gốc cộng thêm lần nữa; trace Submit loại do ID trùng |
| 7: T05, owner trống, 2 | Bỏ do thiếu owner |

Trace Stage 03 Submit sau sửa tính Lan **9 giờ**, Minh **3 giờ**; hai dòng lỗi là dòng 5 và 7, dòng 6 bị loại do trùng `T02`. Trace Stage 03 gốc mới nhất chạy Bash thành công nhưng script trong trace vẫn tính Lan **14 giờ**, Minh **3 giờ**, do cộng dòng trùng.

Trace trước sửa tại `stage-03-bash/traces/20261007-223338_cfa746d5_turn01_8bb8c8f8.jsonl` và `stage-03-bash-Submit/traces/20261007-223336_6c80aa62_turn01_ecb7135f.jsonl` ghi `python --version` exit 1 với RPC `Bash/Service/0x80007072c`. Trace Submit sau sửa `20261007-224635_6c80aa62_turn02_b59ea364.jsonl` chạy thành công, Python 3.14.8; trace tính CSV là `20261007-224825_6c80aa62_turn03_2a81cebd.jsonl`.

Bản sửa Stage 03 ưu tiên Git Bash tại `Program Files/Git/bin/bash.exe` trên Windows (có thể cấu hình `GIT_BASH_EXE`), dùng file tạm cho stdout/stderr và dừng process tree khi timeout. Kiểm tra `uv run pytest tests/test_bash.py`: **6 passed** ở mỗi bản. Suite đầy đủ Submit có **46 passed, 2 failed** vì các test symlink không có quyền Windows (`WinError 1314`), không liên quan Bash.

## Stage 04 - Script và skill

Trong `stage-04-script-skill-Submit/`:

- `workspace/skills/csv-quality/scripts/check_csv.py`: yêu cầu `--max-hours` hữu hạn, không âm; cộng giờ từ dòng hợp lệ; giữ quyền ID ở lần gặp đầu kể cả khi dòng đó sai; sắp lý do loại ổn định; quá tải khi tổng **lớn hơn** ngưỡng.
- `workspace/skills/csv-quality/SKILL.md`: yêu cầu hỏi ngưỡng nếu người dùng chưa nêu và truyền ngưỡng vào lệnh.
- `workspace/skills/csv-quality/references/report-template.md`: bổ sung ngưỡng, tổng, owner quá tải và dòng bị loại cùng lý do.
- Ba file skill trên được đồng bộ vào `fixtures/skills/csv-quality/`.
- `workspace/data/workload.csv` là CSV chính; `workspace/data/workload-edge.csv` kiểm tra ID đầu tiên có `hours=abc`.
- `tests/test_check_csv.py` kiểm tra hồi quy trường hợp ID xuất hiện đầu sai dữ liệu vẫn chặn dòng trùng sau đó.
- `tools/bash.py` dùng runner Windows đã kiểm chứng ở Stage 03 Submit: ưu tiên Git Bash, hỗ trợ `GIT_BASH_EXE`, file tạm cho stdout/stderr và dừng process tree khi timeout. README hướng dẫn cấu hình Windows.

### Trace Stage 04

Trace Stage 04 gốc `stage-04-script-skill/traces/20261007-225716_62f87d86_turn01_eef8898e.jsonl` gọi Bash thất bại với RPC `Bash/Service/0x80007072c`. Trace Submit cũ `stage-04-script-skill-Submit/traces/20261007-225718_50db6963_turn01_5da84514.jsonl` cũng gặp lỗi đó. Sau khi sửa runner, sáu lượt mới đều chạy được:

| Lượt | Nội dung và kết quả |
|---|---|
| Turn 01 | Ngưỡng 8: exit 0; Lan 9, Minh 3; chỉ Lan quá tải; loại dòng 5 `invalid_hours`, dòng 6 `duplicate_id`, dòng 7 `missing_owner`; ghi `output/csv-quality.md` thành công. |
| Turn 02 | Ngưỡng 9: không có owner quá tải vì Lan bằng ngưỡng; tổng và các dòng loại đúng. |
| Turn 03 | Không có ngưỡng: agent hỏi lại, chưa gọi Bash và không tự đoán. |
| Turn 04 | Ngưỡng 7: Lan quá tải, Minh không quá tải. |
| Turn 05 | CSV biên, ngưỡng 0: E01 dòng 2 bị loại do giờ sai; E01 dòng 3 bị loại do ID trùng; Minh 0 giờ, không quá tải. |
| Turn 06 | File `data/missing.csv`: Bash exit 1, stderr báo không đọc được file; agent báo lỗi, không tạo số liệu giả. |

Các trace trên nằm trong `stage-04-script-skill-Submit/traces/`, từ `20261007-230304_30596dc3_turn01_0eb17ed6.jsonl` đến `20261007-230625_30596dc3_turn06_39dd383b.jsonl`. Một số chuỗi tiếng Việt trong stdout/stderr bị mojibake; JSON, exit code và dữ liệu số vẫn đọc được.

### Kết quả kiểm tra

| Kiểm tra | Kết quả |
|---|---|
| `uv run pytest tests/test_check_csv.py` Stage 04 Submit | **PASS** — 13 passed |
| `uv run pytest tests/test_bash.py tests/test_check_csv.py` sau cập nhật runner | **PASS** — 19 passed |
| CSV chính, ngưỡng 8 | **PASS** — Lan 9, Minh 3; chỉ Lan quá tải; dòng 5/6/7 bị loại đúng lý do |
| CSV chính, ngưỡng 9 | **PASS** — không ai quá tải vì Lan bằng ngưỡng |
| CSV biên, ngưỡng 0 | **PASS** — không cộng dòng trùng E01 sau dòng đầu lỗi |
| Thiếu `--max-hours` | **PASS** — argparse từ chối trước khi phân tích |
| File đầu vào không tồn tại | **PASS** — exit khác 0, báo lỗi trên stderr, không trả JSON thành công |
| Trace Stage 04 gốc trước sửa | **FAIL** — lỗi RPC khi chạy Bash |
| Trace Stage 04 Submit sau sửa | **PASS** — sáu lượt hoàn thành; lượt phân tích chính ghi báo cáo thành công |

JSON CSV chính có `row_count=6`, `missing_owner_count=1`, `invalid_hours_count=1`, `duplicate_id_count=1`, `duplicate_ids=["T02"]`. Thống kê chất lượng xét mọi dòng dữ liệu, kể cả dòng không được cộng giờ.

## Ranh giới giữa script và model

Script đọc CSV, kiểm tra số trường/ID/owner/hours, loại dòng, tính tổng và so sánh với ngưỡng. Model cần hỏi lại khi thiếu ngưỡng, diễn giải JSON thành câu trả lời và điền báo cáo. Cập nhật script cần đi kèm cập nhật skill và template để model truyền đủ tham số và báo cáo đúng các trường mới.

## Bằng chứng trong repository

- Stage 03 Bash: `stage-03-bash-Submit/tools/bash.py`, `stage-03-bash/tools/bash.py`.
- Stage 03 CSV: `stage-03-bash-Submit/workspace/data/workload.csv`.
- Stage 04 CSV: `stage-04-script-skill-Submit/workspace/data/workload.csv`, `stage-04-script-skill-Submit/workspace/data/workload-edge.csv`.
- Stage 04 skill/script: `stage-04-script-skill-Submit/workspace/skills/csv-quality/` và `stage-04-script-skill-Submit/tests/test_check_csv.py`.
- Trace Stage 04 gốc: `stage-04-script-skill/traces/20261007-225716_62f87d86_turn01_eef8898e.jsonl`; trace Submit trước sửa runner: `stage-04-script-skill-Submit/traces/20261007-225718_50db6963_turn01_5da84514.jsonl`; sáu trace Submit sau sửa runner nằm trong `stage-04-script-skill-Submit/traces/`.
