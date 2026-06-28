# /.system — Live System State Files

## Purpose
`.system` is the runtime state directory. It holds files that the agent OS reads and writes during active execution — things that change frequently and need to be queryable fast. This is distinct from memory (which is for long-term learning) and `/system/` (which is for architecture documentation).

## What Goes Here
- `agent.db` — SQLite database; in-memory index cache for episodic log, skills, and preferences (gitignored)
- `session-lock.json` — written at session start, deleted at session end; prevents two sessions running simultaneously
- `active-goal.json` — the current active goal being processed (cleared when goal completes)
- `queue.json` — ordered list of tasks currently in the pipeline (written by orchestrator, read by workers)

## What Does NOT Go Here
- Memory files — those live in `/memory/`
- Task definition files — those live in `/tasks/`
- Logs — those live in `/logs/`
- Documentation — that lives in `/system/`

## Gitignore Status
```
/.system/agent.db     ← GITIGNORED (regeneratable)
/.system/session-lock.json  ← GITIGNORED (runtime state)
/.system/active-goal.json   ← COMMITTED (session continuity)
/.system/queue.json         ← COMMITTED (session continuity)
```

## Current State
v1: Directory created. Files are created at runtime by the orchestrator. `agent.db` is already in `.gitignore`.

## Rules
1. `agent.db` is ALWAYS regeneratable from JSONL files — if it is corrupted, run `scripts/rebuild-db.py`
2. `session-lock.json` must be cleaned up at session end — if it exists at session start, it means the last session did not end cleanly; investigate before proceeding
3. Files in `.system/` that are committed to git must be kept small (< 100KB) — large state belongs in JSONL files
4. Never manually edit `.system/` files during an active session — only the orchestrator writes here
