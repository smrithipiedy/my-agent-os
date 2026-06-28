# Momentum Queues
**System:** my-agent-os
**Updated:** Session 1, 2026-06-28

---

## NOW
*What is happening in this session right now*

- Bootstrapping the repository structure from a blank slate (only `AGENTS.md` existed)
- Writing all five foundational system files: `capability-matrix.md`, `implementation-contract.md`, `momentum.md`, `milestones.md`
- Writing session continuity files: `status.md`, `handoff.md`
- Committing everything to git so Session 2 starts with a clean, verified baseline

---

## NEXT
*The next concrete things ready to run after this session*

These are unblocked and can start immediately in Session 2:

1. **Create the inbox system** — `/inbox/goal.md` template + schema for submitting goals
2. **Create the task schema** — define the JSON structure for `/tasks/active/<id>.json` (fields: id, goal_id, description, worker_type, inputs, expected_output, status, created_at, completed_at)
3. **Build the orchestrator skeleton** — a Python script at `/orchestrator/run.py` that reads `goal.md`, calls Gemini API to decompose the goal, and writes task JSON files
4. **Create the worker registry** — `/workers/registry.json` mapping `worker_type` strings to handler modules
5. **Stub the first worker** — `/workers/code_writer.py` that accepts a task dict and returns a result string
6. **Create the memory schema** — define the JSONL structure for `/memory/episodic-log.jsonl`
7. **Create the session log template** — `/logs/session-template.md` that every session log copies

---

## BLOCKED
*Anything waiting on missing info or capability*

- **Gemini API key not confirmed in `.env.local`** — Session 2 cannot run the orchestrator until Smrithi confirms the key is present and accessible. Action: Smrithi creates `.env.local` with `GEMINI_API_KEY=...` before Session 2 starts.
- **Python environment not verified** — need to confirm `python --version` and whether `google-generativeai` package is installed. Action: Session 2 runs `python --version` as first shell check.
- **Supabase credentials not provided** — memory persistence via Supabase deferred until credentials are in `.env.local`. Not blocking v1 (files are the fallback).
- **Worker design decision open** — should workers be Python scripts, Node.js modules, or shell scripts? Stack preference is Python for orchestration, JS for UI. Decision: Python for all v1 workers (aligns with Gemini Python SDK).

---

## IMPROVE
*Self-improvement work to do later (not now)*

- Add a structured `worker_type` validator so unrecognized types fail fast with a clear error instead of silently doing nothing
- Build a retry policy for failed tasks: first retry is immediate, second retry adds a human checkpoint
- Add a diff-based memory deduplication step: before writing to `learnings.md`, check if the same learning already exists (semantic similarity via Gemini embeddings)
- Replace `.env.local` secret management with a proper secrets schema doc + validation at session start
- Build a `/skills/` registry where reusable code patterns are stored and indexed — reduces token cost by reusing proven patterns
- Add a web dashboard (Next.js) to visualize `/logs/` and `/memory/` as Milestone 2 UI

---

## RECURRING
*Things that should run after every session*

1. **Update `handoff.md`** — always the last thing written before ending a session
2. **Update `status.md`** — set `current_session`, `last_action`, `next_action`, `system_health`
3. **Append to `/memory/episodic-log.jsonl`** — one entry per verified task completed this session
4. **Write `/logs/session-YYYYMMDD.md`** — full audit trail of every action taken this session
5. **Append to `/memory/learnings.md`** — at least one learning per session
6. **Run `git add . && git commit -m "Session N: <summary>"`** — commit all changes; repo is the persistent store
7. **Check `blocked` queue** — confirm whether any blockers from the previous session were resolved
