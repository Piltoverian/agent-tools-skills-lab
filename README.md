# Agent Tools & Skills Lab

Lab gồm 5 LangChain agent có giao diện chat Streamlit, giúp quan sát cách model dùng tool, nạp skill và chạy script. Đi từ Stage 00 đến Stage 04 để so sánh khả năng và context gửi model ở từng bước.

Trong repo, các stage dùng chung môi trường Python `.venv` và lockfile `uv.lock` tại thư mục gốc. Mỗi stage có `.env`, dữ liệu làm việc và trace riêng; cũng có thể được sao chép ra ngoài lab để chạy độc lập.

## Lộ trình các stage

| Stage | Khả năng mẫu | Tools | Skills mẫu |
|---|---|---|---|
| [00 — Chat](stage-00-chat/README.md) | Chat với model, chưa truy cập file | Không có | Không có |
| [01 — File tools](stage-01-files/README.md) | Đọc file và ghi báo cáo trong workspace | `read_file`, `write_file` | Không có |
| [02 — Skills](stage-02-skills/README.md) | Nạp hướng dẫn và reference khi cần | `read_file`, `write_file` | `weekly-report` |
| [03 — Bash](stage-03-bash/README.md) | Chạy lệnh, xử lý dữ liệu bằng Python qua Bash | `read_file`, `write_file`, `bash` | `weekly-report` |
| [04 — Script skill](stage-04-script-skill/README.md) | Dùng script kiểm tra CSV được đóng gói trong skill | `read_file`, `write_file`, `bash` | `weekly-report`, `csv-quality` |

Ở Stage 02–04, system prompt ban đầu chỉ chứa catalog gồm tên, mô tả và vị trí skill. Model dùng `read_file` để nạp `SKILL.md` và reference; nội dung được giữ trong history cho các lượt sau. Stage 04 giữ nguyên bộ tool của Stage 03, bổ sung script và hướng dẫn qua skill.

Xem [STAGE-DIFFS.md](STAGE-DIFFS.md) để đối chiếu các file và hàm thay đổi giữa hai stage liên tiếp.

## Cài đặt và chạy

### 1. Chuẩn bị

- Python **3.11+** và [uv](https://docs.astral.sh/uv/).
- API key và tên model hỗ trợ **native tool calling**. Có thể dùng OpenAI hoặc endpoint tương thích OpenAI Chat Completions (`tools`, `tool_calls`, message role `tool`).
- Stage 03–04 cần Bash. Trên Windows, xem hướng dẫn môi trường bên dưới trước khi chạy các stage này.

Từ thư mục gốc `agent-tools-skills-lab`, cài dependencies cho cả 5 stage:

```bash
uv sync --all-packages --locked
```

`uv sync` và `uv run` tự nhận diện workspace chung khi chạy trong một stage. Không tạo `.venv` riêng tại từng stage trong repo.

### 2. Cấu hình model cho stage

Tạo `.env` từ `.env.example` **trong thư mục stage muốn chạy**, rồi điền các biến:

| Biến | Yêu cầu | Nội dung |
|---|---|---|
| `OPENAI_API_KEY` | Bắt buộc | API key của provider |
| `MODEL_NAME` | Bắt buộc | Tên model tại provider/endpoint đã chọn |
| `OPENAI_BASE_URL` | Tùy chọn | URL endpoint tương thích OpenAI nếu dùng provider khác |

Thiếu key hoặc tên model, UI hiện cảnh báo và khóa ô chat. Không đưa `.env` hoặc API key vào bài nộp.

### 3. Chạy ứng dụng

Ví dụ chạy Stage 02 từ thư mục gốc; thay tên thư mục để chạy stage khác.

**PowerShell:**

```powershell
Set-Location stage-02-skills
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
# Mở .env và điền cấu hình trước khi chạy.
uv run streamlit run app.py
```

**Bash (Linux, WSL2 hoặc Git Bash):**

```bash
cd stage-02-skills
test -f .env || cp .env.example .env
# Mở .env và điền cấu hình trước khi chạy.
uv run streamlit run app.py
```

Mở **Local URL** in trong terminal, mặc định `http://localhost:8501`. Chạy từ thư mục stage để Streamlit đọc đúng `.streamlit/config.toml` (theme, chế độ headless, tắt thống kê sử dụng). Nếu chạy nhiều stage cùng lúc, chọn cổng khác, ví dụ:

```bash
uv run streamlit run app.py --server.port 8502
```

### Bash trên Windows

- **Stage 03:** tool tự ưu tiên Git Bash ở `C:\Program Files\Git\bin\bash.exe`. Đặt biến môi trường `GIT_BASH_EXE` trước khi chạy app nếu Git Bash ở nơi khác; nếu không có Git Bash, cài uv và chạy stage bên trong WSL2. Xem [README Stage 03](stage-03-bash/README.md).
- **Stage 04:** theo [README Stage 04](stage-04-script-skill/README.md), dùng môi trường POSIX có Bash; trên Windows, cài uv và chạy stage bên trong WSL2.

Tool `bash` chạy với thư mục hiện tại là `workspace/`, timeout 10 giây và môi trường tối thiểu. **Bash không phải sandbox:** lệnh có quyền truy cập ngoài workspace theo quyền của người chạy app. Dùng dữ liệu giả trong môi trường lab; ứng dụng dành cho một người dùng local.

## Bài tập thực hành

Mở các guide HTML trong trình duyệt. Làm bài trên bản sao stage theo hướng dẫn; bảng lộ trình phía trên mô tả 5 stage mẫu.

| Guide | Stage | Việc cần làm |
|---|---|---|
| [Block 1 — Tra cứu chính sách đúng phiên bản](lab-guide/exercise-block-1.html) | 00 → 02 | Quan sát giới hạn Stage 00, thêm tool `list_files` ở Stage 01/02 và viết skill `refund-policy` để chọn chính sách theo ngày mua. |
| [Block 2 — Kiểm tra quá tải theo người](lab-guide/exercise-block-2.html) | 03 → 04 | Tính tổng giờ bằng Python qua Bash ở Stage 03; mở rộng script `csv-quality`, skill và reference ở Stage 04 để nhận ngưỡng, loại dòng lỗi và không cộng trùng công việc. |

Mỗi block có dữ liệu mẫu, quy tắc xử lý, trường hợp kiểm tra và yêu cầu nộp bài. Đối chiếu câu trả lời với kết quả tool, JSON, file đầu ra và trace theo từng trường hợp.

## Quan sát kết quả

- **Các bước thực hiện:** xem số model/tool calls, arguments, kết quả và lỗi của từng tool.
- **State & Context:** xem tools được cấp, catalog, skill đã được đọc, message history và snapshot ngay trước mỗi model call. Snapshot ghi request ở lớp LangChain.
- **File đầu ra** (Stage 01–04): xem và tải file trong `workspace/output/`. Mở file trên UI không đưa nội dung vào context model.
- **Trace:** mỗi stage ghi event vào `traces/*.jsonl`; lỗi không mong đợi được ghi thêm vào `traces/debug.log`.

Nút **Cuộc trò chuyện mới** xóa history, counters, snapshots và skill đã nạp của hội thoại; file output và trace vẫn được giữ. Xem README từng stage để thử các prompt mẫu và hiểu giới hạn thực thi.

## Tests và dữ liệu workspace

Trong thư mục stage cần kiểm tra, chạy tests offline với mock model, không cần API key:

```bash
uv run pytest
```

[verification.md](verification.md) ghi kết quả kiểm tra ngày **2026-10-04**, gồm tests và UI với mock endpoint. Đây là snapshot của môi trường lúc kiểm tra: tài liệu này còn mô tả `.venv`/lockfile riêng và chưa xác nhận live model hay Windows/WSL2/Linux. Setup hiện tại trong repo dùng workspace chung như hướng dẫn phía trên.

Stage 01–04 khởi tạo `workspace/` từ `fixtures/` nếu workspace chưa tồn tại. Để khôi phục dữ liệu gốc, chạy trong thư mục stage:

```bash
uv run python reset_workspace.py
```

Reset cần marker `.lab-workspace`, khôi phục nội dung từ fixtures và xóa output; traces được giữ. Nếu đang làm bài bằng cách sửa skill trong `workspace/`, đồng bộ thay đổi cần giữ sang `fixtures/` trước khi reset.

## Chạy một stage độc lập

Sao chép thư mục stage ra **ngoài workspace của repo**, không kèm `.venv`, rồi chạy `uv sync` trong bản sao để tạo môi trường và lockfile riêng. Ví dụ trong Bash:

```bash
cp -R stage-02-skills ~/my-agent-lab
cd ~/my-agent-lab
uv sync
test -f .env || cp .env.example .env
# Điền cấu hình model trong .env của bản sao.
uv run streamlit run app.py
```

Với Stage 03–04, bản sao vẫn cần môi trường Bash phù hợp như README tương ứng.
