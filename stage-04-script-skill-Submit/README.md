# Stage 04: Script skill

read_file, write_file, bash + skill catalog (weekly-report, csv-quality cÃ³ script).

- Tools: `read_file(path)`, `write_file(path, content)`, `bash(command)`
- Skills: `weekly-report`, `csv-quality`

## CÃ i Ä‘áº·t vÃ  cháº¡y

Cáº§n Python 3.11+ vÃ  [uv](https://docs.astral.sh/uv/). Cháº¡y trong thÆ° má»¥c project nÃ y:

```bash
cd stage-04-script-skill
uv sync --all-packages --locked
cp -n .env.example .env
# Äiá»n OPENAI_API_KEY, MODEL_NAME (vÃ  OPENAI_BASE_URL náº¿u dÃ¹ng endpoint khÃ¡c) vÃ o .env
uv run streamlit run app.py
```

Trong lab, cáº£ 5 stage dÃ¹ng chung `.venv` vÃ  `uv.lock` á»Ÿ thÆ° má»¥c `agent-tools-skills-lab`; `uv sync` vÃ  `uv run` tá»± nháº­n diá»‡n workspace. KhÃ´ng táº¡o `.venv` riÃªng táº¡i stage. Náº¿u copy stage ra ngoÃ i lab Ä‘á»ƒ cháº¡y Ä‘á»™c láº­p, dÃ¹ng `uv sync` thay cho lá»‡nh sync trÃªn.

Má»Ÿ **Local URL** in ra terminal (máº·c Ä‘á»‹nh `http://localhost:8501`). `.streamlit/config.toml` cá»§a project báº­t `server.headless`, nÃªn Streamlit khÃ´ng há»i email á»Ÿ láº§n cháº¡y Ä‘áº§u vÃ  khÃ´ng tá»± má»Ÿ trÃ¬nh duyá»‡t; file nÃ y cÅ©ng Ä‘áº·t mÃ u theme vÃ  táº¯t gá»­i thá»‘ng kÃª sá»­ dá»¥ng. Streamlit chá»‰ Ä‘á»c file nÃ y khi cháº¡y trong thÆ° má»¥c project.

ÄÆ°á»ng dáº«n Ä‘Æ°á»£c resolve tá»« vá»‹ trÃ­ source. Cháº¡y tá»« thÆ° má»¥c khÃ¡c: `uv run --project stage-04-script-skill streamlit run stage-04-script-skill/app.py`; app váº«n dÃ¹ng Ä‘Ãºng `.env`, workspace vÃ  `traces/` cá»§a project nÃ y.

## Cáº¥u hÃ¬nh model

| Biáº¿n | Báº¯t buá»™c | Ã nghÄ©a |
|---|---|---|
| `OPENAI_API_KEY` | CÃ³ | API key cá»§a provider |
| `MODEL_NAME` | CÃ³ | TÃªn model, cáº§n há»— trá»£ native tool calling |
| `OPENAI_BASE_URL` | KhÃ´ng | OpenAI-compatible endpoint. Endpoint pháº£i há»— trá»£ tool calling theo chuáº©n OpenAI Chat Completions (`tools`, `tool_calls`, message role `tool`). |

Thiáº¿u `OPENAI_API_KEY` hoáº·c `MODEL_NAME`: UI hiá»‡n cáº£nh bÃ¡o, Ã´ chat bá»‹ khÃ³a vÃ  app khÃ´ng gá»i model. App khÃ´ng truyá»n `temperature` hay tham sá»‘ model khÃ¡c; key khÃ´ng Ä‘Æ°á»£c ghi vÃ o trace/log.

## Prompt thá»­

1. `Kiá»ƒm tra cháº¥t lÆ°á»£ng data/tasks.csv vÃ  ghi bÃ¡o cÃ¡o vÃ o output/csv-quality.md.`
2. `Kiá»ƒm tra cháº¥t lÆ°á»£ng data/khong-co.csv.`
3. (LÆ°á»£t tiáº¿p theo) `DÃ²ng nÃ o thiáº¿u owner?`

Ká»³ vá»ng: Agent Ä‘á»c `skills/csv-quality/SKILL.md`, cháº¡y `python skills/csv-quality/scripts/check_csv.py --input data/tasks.csv` báº±ng bash, kiá»ƒm tra exit code, dÃ¹ng JSON (6 dÃ²ng, 1 thiáº¿u owner, 1 hours sai, T02 láº·p), Ä‘á»c reference khi viáº¿t bÃ¡o cÃ¡o, ghi `output/csv-quality.md`. File khÃ´ng tá»“n táº¡i: script exit 1, agent bÃ¡o lá»—i thá»±c thi, khÃ´ng bá»‹a thá»‘ng kÃª. CSV gá»‘c khÃ´ng bá»‹ sá»­a.

## Xem output, trace, state vÃ  context

- **Bá»‘ cá»¥c**: trang khÃ´ng cuá»™n; chá»‰ cá»™t chat vÃ  cá»™t State & Context cuá»™n riÃªng, cao theo cá»­a sá»• trÃ¬nh duyá»‡t (khá»‘i CSS nhá» `PAGE_CSS` trong `app.py`). Ná»™i dung dÃ i (trace, báº£ng, file output) hiá»‡n Ä‘áº§y Ä‘á»§ trong cá»™t, khÃ´ng cÃ³ khung cuá»™n lá»“ng nhau.
- **Chat (cá»™t trÃ¡i)**: má»—i lÆ°á»£t assistant cÃ³ expander **CÃ¡c bÆ°á»›c thá»±c hiá»‡n (N model call, M tool call)**: tá»«ng model call, má»—i tool call trong má»™t khung cÃ³ badge ok/lá»—i, thá»i gian cháº¡y, arguments (lá»‡nh bash hiá»‡n dáº¡ng code), result; content/stdout/stderr hiá»‡n Ä‘áº§y Ä‘á»§. Final answer náº±m ngoÃ i expander. NÃºt **Cuá»™c trÃ² chuyá»‡n má»›i** (cáº¡nh tiÃªu Ä‘á») xÃ³a history, counters, snapshots, skill Ä‘Ã£ load cá»§a conversation cÅ©; khÃ´ng xÃ³a `traces/` hay file output.
- **State & Context (cá»™t pháº£i)**: badge tráº¡ng thÃ¡i vÃ  4 counters cáº­p nháº­t trong lÃºc cháº¡y (rÃª chuá»™t vÃ o biá»ƒu tÆ°á»£ng ? Ä‘á»ƒ xem Ä‘á»‹nh nghÄ©a); **Tools Ä‘Æ°á»£c cáº¥p** (schema); **Skills trong catalog** (metadata, chÆ°a pháº£i Ä‘Ã£ load); **Skill content Ä‘Ã£ vÃ o history** (chá»‰ khi `read_file` SKILL.md thÃ nh cÃ´ng, kÃ¨m message index vÃ  tool_call_id); **TÃ i nguyÃªn Ä‘Ã£ Ä‘á»c**; **Message history hiá»‡n táº¡i**; **Event log**.
- **Context gá»­i model (lá»›p LangChain)**: snapshot request chá»¥p ngay trÆ°á»›c má»—i model call: system prompt thá»±c táº¿, messages theo thá»© tá»±, tools schema, metadata (conversation_id, run_id, lÆ°á»£t, model call, event sequence, thá»i Ä‘iá»ƒm, model name, sá»‘ message, sá»‘ kÃ½ tá»±, usage náº¿u provider tráº£ vá»). TrÆ°á»›c model call Ä‘áº§u tiÃªn, panel hiá»‡n **Context cáº¥u hÃ¬nh (chÆ°a gá»­i)**. Sau khi lÆ°á»£t káº¿t thÃºc cÃ³ thá»ƒ chá»n láº¡i snapshot cÅ© (chá»‰ Ä‘á»c, khÃ´ng cháº¡y láº¡i agent) vÃ  táº£i snapshot JSON. Snapshot khÃ´ng pháº£i provider wire payload, tokenizer output hay ná»™i dung bÃªn trong model.
- **File Ä‘áº§u ra**: chá»n file trong `workspace/output/`, xem Markdown/text vÃ  táº£i xuá»‘ng. Xem file lÃ  thao tÃ¡c UI, khÃ´ng gá»­i ná»™i dung vÃ o context cá»§a model.
- **Trace trÃªn Ä‘Ä©a**: `traces/<thá»i gian>_<conversation>_turnNN_<run>.jsonl`, má»—i dÃ²ng má»™t event (sequence, type, tool_call_id, tool_name, arguments, result, elapsed_ms; event `model_request` kÃ¨m snapshot request). Exception khÃ´ng mong Ä‘á»£i ghi thÃªm vÃ o `traces/debug.log`.

## Äá»‹nh nghÄ©a trong State & Context

| TrÆ°á»ng | Ã nghÄ©a |
|---|---|
| LÆ°á»£t chat | Má»™t láº§n gá»­i input; báº¯t Ä‘áº§u tá»« 1 trong má»—i conversation. |
| Láº§n gá»i model trong lÆ°á»£t | TÄƒng ngay trÆ°á»›c má»—i model request thá»±c táº¿ (middleware `wrap_model_call`); reset khi gá»­i lÆ°á»£t má»›i. Retry ná»™i bá»™ cá»§a OpenAI SDK khÃ´ng quan sÃ¡t Ä‘Æ°á»£c nÃªn khÃ´ng Ä‘áº¿m riÃªng. |
| BÆ°á»›c sá»± kiá»‡n trong lÆ°á»£t | Sá»‘ thá»© tá»± observer event (`user_submitted`, `model_request`, `model_response`, `tool_started`, `tool_finished`, `run_completed`, `run_failed`). KhÃ´ng pháº£i LangGraph step. |
| Tool calls trong lÆ°á»£t | Sá»‘ tool call Ä‘Ã£ thá»±c thi, má»—i call cÃ³ ID riÃªng. Má»™t model response cÃ³ thá»ƒ yÃªu cáº§u nhiá»u tool calls. |
| Tráº¡ng thÃ¡i | Sáºµn sÃ ng / Äang gá»i model / Äang cháº¡y tool / HoÃ n táº¥t / Lá»—i. |

## Giá»›i háº¡n thá»±c thi

Má»—i lÆ°á»£t: tá»‘i Ä‘a 8 láº§n gá»i model (`ModelCallLimitMiddleware`), 20 tool calls (`ToolCallLimitMiddleware`), recursion limit LangGraph 50 (sá»‘ bÆ°á»›c graph, khÃ´ng pháº£i sá»‘ tool calls). GiÃ¡ trá»‹ náº±m trong `config.py`.

Lá»—i runtime/provider: UI hiá»‡n lá»—i, giá»¯ trace Ä‘Ã£ cÃ³, rollback model history vá» trÆ°á»›c lÆ°á»£t lá»—i. App khÃ´ng tá»± cháº¡y láº¡i lÆ°á»£t lá»—i.

## Workspace vÃ  reset

Agent chá»‰ lÃ m viá»‡c trong `workspace/`: `read_file` Ä‘á»c file trong workspace, `write_file` chá»‰ ghi dÆ°á»›i `workspace/output/`. Láº§n cháº¡y Ä‘áº§u, náº¿u chÆ°a cÃ³ workspace, app copy tá»« `fixtures/`; rerun khÃ´ng ghi Ä‘Ã¨.

KhÃ´i phá»¥c dá»¯ liá»‡u gá»‘c (xÃ³a output, giá»¯ traces):

```bash
uv run python reset_workspace.py
```

Lá»‡nh chá»‰ cháº¡y khi workspace cÃ³ marker `.lab-workspace` cá»§a lab.

## Skills

`skill_catalog.py` quÃ©t `workspace/skills/*/SKILL.md`, chá»‰ Ä‘Æ°a `name`, `description`, `location` vÃ o system prompt. Body vÃ  reference khÃ´ng cÃ³ trong context ban Ä‘áº§u. Model tá»± Ä‘á»c SKILL.md báº±ng `read_file` khi task khá»›p description; tool result á»Ÿ láº¡i trong history cho cÃ¡c lÆ°á»£t sau. Skill thiáº¿u metadata hoáº·c YAML sai bá»‹ bá» qua kÃ¨m diagnostic; trÃ¹ng `name` bÃ¡o lá»—i vÃ  giá»¯ báº£n Ä‘áº§u (xem expander **Skills trong catalog**). ThÃªm skill má»›i: táº¡o `workspace/skills/<name>/SKILL.md` (vÃ  báº£n trong `fixtures/skills/` náº¿u muá»‘n giá»¯ sau reset).

## Tests

```bash
uv run pytest
```

Tests dÃ¹ng mock model, khÃ´ng cáº§n API key.

## Giá»›i háº¡n mÃ´i trÆ°á»ng

- **Bash khÃ´ng pháº£i sandbox.** `bash` cháº¡y `bash -c` vá»›i cwd lÃ  `workspace/`, env tá»‘i thiá»ƒu (PATH trá» Python trong `.venv` cá»§a project, locale UTF-8, khÃ´ng cÃ³ API key), timeout 10 giÃ¢y. Path check cá»§a `read_file`/`write_file` khÃ´ng Ã¡p dá»¥ng cho Bash: lá»‡nh váº«n Ä‘á»c/ghi Ä‘Æ°á»£c ngoÃ i workspace vá»›i quyá»n user cháº¡y app.

- Chá»‰ cháº¡y trong mÃ´i trÆ°á»ng lab vá»›i dá»¯ liá»‡u giáº£. KhÃ´ng mount thÆ° má»¥c cÃ¡ nhÃ¢n hoáº·c secrets vÃ o mÃ¡y/container cháº¡y lab.

- Cần môi trường có `bash`. Trên Windows, tool ưu tiên Git Bash tại `Program Files/Git/bin/bash.exe`; đặt `GIT_BASH_EXE` nếu Git Bash ở vị trí khác. Nếu không cài Git Bash, dùng WSL2 và chạy project bên trong WSL2.

- Cháº¡y local, má»™t ngÆ°á»i dÃ¹ng cho má»—i project. KhÃ´ng cÃ³ xÃ¡c thá»±c, khÃ´ng chia sáº» workspace Ä‘a ngÆ°á»i dÃ¹ng.
