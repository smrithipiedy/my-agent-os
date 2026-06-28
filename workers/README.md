# /workers — Worker Processes

## Purpose
Workers are the execution units of the agent OS. Each worker accepts a task dict, performs one specific type of work, and writes its output to `/outputs/<task-id>/result.md`. Workers are stateless — they read from their input task and write to their output location. They do not communicate with each other.

## What Goes Here
- One Python file per worker type
- `registry.json` — maps `worker_type` strings to handler file paths
- `base_worker.py` — abstract base class all workers inherit from
- Worker-specific utility modules (e.g., `_gemini_client.py`, `_shell_utils.py`)

## Worker Types (v1)

| worker_type | File | What It Does |
|---|---|---|
| `code_writer` | `code_writer.py` | Calls Gemini to generate code for a task; writes to result.md |
| `shell_runner` | `shell_runner.py` | Executes PowerShell commands after human approval; writes stdout/stderr to result.md |
| `web_searcher` | `web_searcher.py` | Searches the web or reads a URL; writes extracted content to result.md |
| `file_editor` | `file_editor.py` | Reads an existing file and applies targeted edits; writes diff or new content to result.md |
| `browser_agent` | `browser_agent.py` | Launches a browser sub-agent for interactive web tasks; writes screenshot path and summary to result.md |

## What Does NOT Go Here
- Orchestration logic — that lives in `/orchestrator/`
- Memory writes — workers write to `/outputs/` only; memory is written by the orchestrator after verification
- Task schema — defined in `/projects/_template/tasks.md`

## Rules
1. Every worker MUST inherit from `base_worker.py` and implement the `execute(task: dict) -> str` method
2. Workers MUST write their output to `/outputs/<task-id>/result.md` — no other output paths
3. Workers MUST NOT write to memory directly
4. Shell commands MUST pause for human approval before executing — use the approval gate in `_shell_utils.py`
5. Workers MUST update the task JSON's `status`, `completed_at`, `tokens_used`, and `attempts` fields after execution
6. A worker that fails MUST write an error to `/outputs/<task-id>/error.md` and set `status: failed`
7. Workers MUST be invoked via the registry — never called directly by name
