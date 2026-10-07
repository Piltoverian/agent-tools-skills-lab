# Block 2 — Analysis and evidence

## Scope and changes

Block 2 extends the Stage 03 Bash exercise with the Stage 04 `csv-quality` script and skill.

- Stage 03 Submit contains `workspace/data/workload.csv` with the six rows specified by the guide. Its Bash runner was updated to prefer Git Bash on Windows, allow `GIT_BASH_EXE`, use file-backed output, and terminate the process tree on timeout: see `stage-03-bash-Submit/tools/bash.py` and `stage-03-bash-Submit/workspace/data/workload.csv`.
- Stage 04 Submit updates `workspace/skills/csv-quality/{SKILL.md,references/report-template.md,scripts/check_csv.py}` and the matching fixture copies. The script requires a finite, nonnegative `--max-hours`; totals only valid rows; reserves each nonempty ID on its first occurrence, even if that row has invalid data; returns stable exclusion reasons; and treats overload as total hours strictly greater than the threshold. Existing all-row quality counts and `issues` remain in the output.
- Stage 04 adds the main CSV, the invalid-first-occurrence edge CSV, and a regression test: `stage-04-script-skill-Submit/workspace/data/workload.csv`, `stage-04-script-skill-Submit/workspace/data/workload-edge.csv`, and `stage-04-script-skill-Submit/tests/test_check_csv.py`.

## Results by requested case

### Stage 03: Bash calculation

Input: `stage-03-bash-Submit/workspace/data/workload.csv`.

- The successful Bash run returns exit code 0, stdout totals `Lan: 9`, `Minh: 3`, and identifies line 5 (`abc` hours), line 7 (empty owner), and line 6 (duplicate `T02`) as excluded. This matches the input and guide. Evidence: `stage-03-bash-Submit/traces/20261007-224825_6c80aa62_turn03_2a81cebd.jsonl`, Stage 03 calculation turn 03.
- The earlier `python --version` calls in `stage-03-bash-Submit/traces/20261007-223336_6c80aa62_turn01_ecb7135f.jsonl` and the first retries in `stage-03-bash-Submit/traces/20261007-224635_6c80aa62_turn02_b59ea364.jsonl` return exit code 1. Their stdout contains the Windows RPC error `Bash/Service/0x80007072c` (UTF-16LE bytes represented with embedded NULs), stderr is empty, and the tool envelope still says `ok: true`. A later retry in turn 02 returns exit code 0 and `Python 3.14.8`. This is a runner/launcher failure, not a CSV calculation result; the Submit runner change addresses Windows shell selection and output handling. The initial failures remain part of the trace evidence.

### Stage 04: threshold 8

`stage-04-script-skill-Submit/traces/20261007-230304_30596dc3_turn01_0eb17ed6.jsonl`, turn 01:

- Bash command: `python skills/csv-quality/scripts/check_csv.py --input data/workload.csv --max-hours 8`.
- Exit code 0; stdout JSON reports totals Lan 9.0 and Minh 3.0, with Lan as the only overloaded owner. Exclusions are line 5 `invalid_hours`, line 6 `duplicate_id`, and line 7 `missing_owner`; quality counts are row_count 6 and one each missing owner, invalid hours, and duplicate ID. Stderr is empty.
- The agent reads the skill and report template, interprets the JSON correctly, and calls `write_file`; the trace records `output/csv-quality.md` created (1674 bytes) and includes its Markdown content.

### Stage 04: threshold 9

`stage-04-script-skill-Submit/traces/20261007-230347_30596dc3_turn02_576cda0f.jsonl`, turn 02:

- Bash command: `python skills/csv-quality/scripts/check_csv.py --input data/workload.csv --max-hours 9`.
- Exit code 0; totals and excluded rows match the threshold 8 case; `overloaded_owners` is empty because Lan's total equals, rather than exceeds, 9. Stderr is empty. The agent reports no overloaded owner.

### Stage 04: threshold omitted

`stage-04-script-skill-Submit/traces/20261007-230407_30596dc3_turn03_226c3486.jsonl`, turn 03, followed by `stage-04-script-skill-Submit/traces/20261007-230416_30596dc3_turn04_dffe1a47.jsonl`, turn 04:

- The first user request omits a threshold. The agent asks the user for the maximum hours and says it has not run the analysis. No Bash call is made in that turn.
- The user then supplies 7; the agent runs with `--max-hours 7`, receives exit code 0, and correctly identifies Lan (9) as overloaded. This confirms the threshold is gathered before the conclusion rather than inferred from prior context.

### Stage 04: first-seen ID has invalid hours

`stage-04-script-skill-Submit/traces/20261007-230514_30596dc3_turn05_f8af4575.jsonl`, turn 05:

- Input: `workspace/data/workload-edge.csv` (`E01,Lan,abc`; then `E01,Lan,5`; then `E02,Minh,0`). Command uses `--max-hours 0` and exits 0.
- JSON has only Minh at 0.0 hours, no overloaded owners, and excludes line 2 for `invalid_hours` and line 3 for `duplicate_id`. The agent's response matches those results; 5 hours are not incorrectly credited to Lan.
- Regression coverage is in `stage-04-script-skill-Submit/tests/test_check_csv.py::test_first_seen_invalid_hours_still_reserves_duplicate_id`.

### Stage 04: missing input file

`stage-04-script-skill-Submit/traces/20261007-230625_30596dc3_turn06_39dd383b.jsonl`, turn 06:

- Bash command: `python skills/csv-quality/scripts/check_csv.py --input data/missing.csv --max-hours 8`.
- Tool call completes with exit code 1, empty stdout, stderr `ERROR: Không đọc được file data/missing.csv: No such file or directory`, and `timed_out: false`. The agent reports the input error and does not return successful totals or create a substitute CSV.

## Tests and report artifact

- `report/block-2.md` records Stage 04 Submit `uv run pytest tests/test_check_csv.py`: 13 passed, and the combined Bash/script check: 19 passed. It records Stage 03 Submit's full suite as 46 passed, 2 failed due to Windows symlink privilege (`WinError 1314`); that suite result is not a clean full-suite pass. Test sources are in `stage-03-bash-Submit/tests/test_bash.py` and `stage-04-script-skill-Submit/tests/test_check_csv.py`. These are the recorded results; this analysis did not rerun the suites.
- The threshold 8 trace proves the agent generated a correct Markdown report, but the traced path is `output/csv-quality.md`. The guide's verification prompt requests `output/workload.md`; the trace therefore does not evidence that exact requested destination. Also, the report file is not present under the current Submit workspace at inspection time, although the trace records a successful `write_file` call. Treat the trace as evidence of the write at that time, not evidence that the artifact remains in the current workspace.
- The verification prompt in the guide asks for a report and trace for threshold 8, threshold 9, omitted threshold, and missing file. The traces cover all four interactions; only the threshold 8 interaction contains a `write_file` call. If a separate report per interaction is expected, it is not evidenced by these traces.

## Conclusion

The direct CSV results, first-seen edge behavior, omitted-threshold clarification, and missing-file handling match the Block 2 rules. The recorded Stage 04 focused tests pass. Stage 03's early Windows RPC failures are real failures despite the tool envelope's `ok: true`; a later retry succeeds after the runner change. The main artifact gap is report destination/persistence: the trace shows `output/csv-quality.md`, not the guide's `output/workload.md`, and the output is absent from the current Submit workspace.
