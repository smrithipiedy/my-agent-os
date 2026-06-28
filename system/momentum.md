# Momentum Queues
**System:** my-agent-os
**Updated:** Session 2, 2026-06-28

---

## NOW
*Session 2 — just completed*

- Created all Part A system docs: REQUIREMENTS.md, WORKFLOW.md, system/FAILURE.md, system/contracts.md, system/knowledge.md, system/memory.md
- Created all 13 Part B canonical folder READMEs
- Created all 8 Part C project template files in /projects/_template/
- Created all 8 Part D real project files in /projects/smrithi-agent-os/
- Updating root continuity files (this file, status.md, handoff.md)
- Committing everything to git

---

## NEXT
*Session 3 — unblocked, ready to run*

1. Run `python --version` — verify Python ≥ 3.10
2. Install `pip install google-generativeai python-dotenv` (with approval)
3. Create `/inbox/goal.md` — goal intake template
4. Create `/tasks/schema.json` — task JSON schema
5. Write `/workers/registry.json` — worker type registry
6. Build `/workers/base_worker.py` — abstract base class
7. Build `/workers/code_writer.py` — first real worker (Gemini API)
8. Build `/orchestrator/run.py` — reads goal, decomposes, presents for approval, dispatches
9. Build `/scripts/setup.py` — environment validator
10. Create `/memory/episodic-log.jsonl` (empty with schema comment)
11. Create `/memory/preferences.md` (Smrithi's confirmed preferences)
12. Update all continuity files; commit

---

## BLOCKED
*Waiting on*

- Python environment not verified — `python --version` needed at Session 3 start
- `google-generativeai` not installed — `pip install` needed at Session 3 (requires approval)
- No executable code yet — Milestone 1 end-to-end run cannot happen until Session 3 builds the orchestrator

---

## IMPROVE
*Self-improvement work for later*

- Add a memory deduplication step to learnings.md: check semantic similarity before appending
- Build a `health-check.py` script that validates system integrity at session start (missing files, orphaned tasks, memory gaps)
- Add a `/rules/` policy checker that verifies tasks satisfy all rules before execution
- Consider adding Smrithi's preferences to the orchestrator prompt so Gemini-generated tasks respect her working style automatically
- Add PATTERN-P003 (JSONL memory format) to `/skills/` once the skills system is built (Milestone 2)

---

## RECURRING
*Run after every session*

1. Update `handoff.md` (root level) — last thing before ending session
2. Update `status.md` (root level) — set current session, last action, next action, system health
3. Update `projects/smrithi-agent-os/status.md` and `handoff.md`
4. Update `system/momentum.md` — clear NOW, populate NEXT from carry-forward
5. Append to `/memory/episodic-log.jsonl` — one entry per verified task
6. Append to `/memory/learnings.md` — at least one learning (once memory files exist)
7. Commit: `git add .; git commit -m "Session N: <summary>"`
8. Check BLOCKED queue — confirm whether any blockers from previous session were resolved
