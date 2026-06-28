# Capability Matrix
**Runtime:** Google Antigravity IDE (Claude Sonnet 4.6 / Gemini backend)
**Machine:** Windows, single user — Smrithi P
**Assessed:** Session 1, 2026-06-28

---

## Capability Assessment

| Capability | Status | Notes |
|---|---|---|
| repo read access | **YES** | Full read via `view_file`, `list_dir`, `grep_search` |
| repo write access | **YES** | Full write via `write_to_file`, `replace_file_content`, `multi_replace_file_content` |
| shell access | **YES (partial)** | `run_command` executes PowerShell; requires user approval for unsafe commands |
| filesystem search | **YES** | `grep_search` + `list_dir` cover full repo tree |
| file editing | **YES** | Inline edits via replace tools; multi-chunk edits supported |
| git access | **YES (partial)** | Git commands via `run_command`; requires user approval for push/remote ops |
| network access | **YES (partial)** | `read_url_content` + `search_web` for reads; no outbound webhooks from agent |
| package install ability | **YES (partial)** | `npm install`, `pip install` via `run_command`; requires user approval each time |
| local database availability | **PARTIAL** | SQLite usable locally; Supabase/Firebase/MongoDB reachable via SDK + credentials |
| browser control | **YES (partial)** | `browser_subagent` for automated browsing; no persistent session |
| screenshot or vision support | **YES** | `browser_subagent` captures screenshots; `generate_image` for synthesis |
| desktop input control | **NO** | No native OS-level mouse/keyboard automation |
| tool-calling support | **YES** | Full MCP tool-calling; sub-agents with `browser_subagent` |
| sub-agent support | **YES** | `browser_subagent` spawns autonomous child agents |
| long-running background execution | **PARTIAL** | `run_command` with async; `command_status` for polling; no daemon persistence across sessions |
| cron or scheduled execution | **NO** | No native scheduler; no always-on process |
| webhook or event trigger support | **NO** | No inbound event listener in this runtime |
| persistent storage | **PARTIAL** | Files in repo are persistent; in-memory state resets per session |
| UI or dashboard rendering | **YES (partial)** | HTML/JS/React via `write_to_file`; `browser_subagent` to preview; no live hot-reload |
| secret management | **PARTIAL** | `.env` files in repo; no secrets vault; user manages API keys manually |
| approval and interruption controls | **YES** | `run_command` requires user approval for unsafe ops; agent can pause and ask |
| multi-machine support | **NO** | Single Windows machine; no distributed execution |

---

## Gap Resolution Plan

| Capability | Status | Resolution |
|---|---|---|
| desktop input control | NO | **Defer safely** — not needed for v1 task orchestration |
| cron / scheduled execution | NO | **Emulate in repo** — a `scheduler/` module will write task files that the user manually triggers, or use Windows Task Scheduler via a setup script |
| webhook / event trigger | NO | **Emulate in repo** — polling loop in a long-running script; external webhooks deferred to v2 |
| multi-machine support | NO | **Narrow scope explicitly** — v1 is single-machine only; multi-machine is an explicit non-goal |
| long-running background exec | PARTIAL | **Emulate in repo** — background tasks run via `run_command` async; state checkpointed to files so sessions can resume |
| secret management | PARTIAL | **Emulate in repo** — `.env.local` file with a documented schema; secrets never committed; user loads them before each session |
| persistent storage (cross-session) | PARTIAL | **Emulate in repo** — all agent state written to `/memory/` and `/logs/` directories in the repo; file system is the persistent store |
| local database | PARTIAL | **Integrate external** — Supabase (already in Smrithi's stack) for structured memory; SQLite as local fallback |
| shell access (approval gate) | PARTIAL | **Narrow scope** — auto-safe commands pre-defined in a safe-list; others always request approval |
| browser control (no persistence) | PARTIAL | **Emulate in repo** — browser tasks write results to `/outputs/`; state captured as files |
