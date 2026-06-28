# /memory — All Memory Layer Files

## Purpose
Memory is the persistence layer that makes the agent OS smarter over time. Everything the system learns, every task it runs, every preference Smrithi states — all of it is stored here. Memory is what transforms a stateless session-by-session tool into a system that accumulates knowledge and improves.

## What Goes Here
- `episodic-log.jsonl` — complete record of every task ever run (append-only)
- `learnings.md` — extracted lessons from completed tasks
- `skills-index.md` — index of all skills in `/skills/`
- `preferences.md` — Smrithi's stated preferences and working style
- `weekly-summary.md` — auto-generated weekly digest

See `/system/memory.md` for the full memory index with types, update triggers, and retrieval methods.

## Memory Types Quick Reference

| File | Type | Temperature | Updated |
|---|---|---|---|
| `episodic-log.jsonl` | Episodic | Cold | After every task |
| `learnings.md` | Semantic | Warm | After verified tasks |
| `skills-index.md` | Procedural | Warm | When skills are added |
| `preferences.md` | Preference | Hot | When Smrithi states a preference |
| `weekly-summary.md` | Temporal | Warm | Every 7 sessions |

## What Does NOT Go Here
- Task files — those live in `/tasks/`
- Worker outputs — those live in `/outputs/`
- Session logs — those live in `/logs/`
- SQLite cache — that lives in `/.system/agent.db` (gitignored)

## Rules
1. `episodic-log.jsonl` is append-only — never edit or delete lines
2. `learnings.md` entries must be concrete and non-duplicate
3. `preferences.md` is the only memory file that can be overwritten (not append-only) — this is how preferences are updated
4. All files in `/memory/` (except `agent.db`) are committed to git
5. If any memory file is corrupted, rebuild from git history — the repo is the backup
6. Memory is never written by workers directly — only the orchestrator writes memory after verification
