# Milestones
**System:** my-agent-os
**Owner:** Smrithi P
**Updated:** Session 1, 2026-06-28

---

## Milestone 1 — Full Loop: Goal → Result → Memory → Human
**Status:** NOT STARTED
**Target:** Session 2–4

### What It Proves
The system can accept a real goal, autonomously break it into tasks, execute each task, verify the output is correct, persist what it learned, and show Smrithi exactly what happened — without Smrithi having to manage the steps manually.

### The Eight Required Steps (all must work end-to-end)

| # | Step | Artifact Produced |
|---|---|---|
| 1 | **Accept a goal** | `/inbox/goal.md` — Smrithi writes a goal in plain language |
| 2 | **Decompose into tasks** | `/tasks/active/<id>.json` — structured task list with worker types and expected outputs |
| 3 | **Route to worker** | worker selected from `/workers/registry.json` based on `worker_type` field |
| 4 | **Execute** | `/outputs/<task-id>/result.md` — worker's raw output |
| 5 | **Verify result** | `/outputs/<task-id>/verification.md` — `verified: true/false` + reason |
| 6 | **Record memory** | `/memory/episodic-log.jsonl` — one JSONL entry per verified task |
| 7 | **Show human** | `/logs/session-YYYYMMDD.md` — full audit log Smrithi can read |
| 8 | **Learn one thing** | `/memory/learnings.md` — at least one concrete lesson appended |

### What It Requires
- `/inbox/goal.md` template and schema
- `/orchestrator/run.py` — reads goal, calls Gemini API, writes task JSON
- `/workers/registry.json` — maps worker_type to handler
- At least one working worker: `code_writer.py`
- Verifier logic (separate from the worker that ran the task)
- `/memory/episodic-log.jsonl` schema and writer
- `/logs/` session log writer
- Gemini API key in `.env.local`
- Python environment with `google-generativeai` installed

### Definition of Done
All eight steps produce their artifacts for a single real goal. The full chain is demonstrated with:
- A committed `/inbox/goal.md` containing the test goal
- A committed `/tasks/active/` directory with at least one task JSON file
- A committed `/outputs/` directory with `result.md` and `verification.md` for each task
- At least one entry in `/memory/episodic-log.jsonl`
- A `/logs/session-YYYYMMDD.md` showing the full run
- At least one entry in `/memory/learnings.md`

### Evidence Required
- All six artifact types exist in the repo after the session
- `verification.md` contains `verified: true` for at least one task
- Session log shows timestamps and file paths touched — not just a summary
- `handoff.md` records the milestone as achieved with commit hash

---

## Milestone 2 — Memory and Skill System
**Status:** NOT STARTED
**Target:** Session 5–8 (after Milestone 1 is done)

### What It Proves
The system remembers what it has done across sessions and reuses proven patterns instead of regenerating everything from scratch every time.

### What It Requires
- `/memory/episodic-log.jsonl` populated with ≥ 5 real task entries from Milestone 1 runs
- `/skills/` directory with at least 3 reusable code/prompt patterns extracted from previous runs
- A skill retriever: given a new task description, searches `/skills/` for relevant patterns (semantic or keyword match)
- A skill writer: after a verified task, extracts the pattern and writes it to `/skills/<slug>.md`
- Memory summary generator: produces `/memory/weekly-summary.md` from the last 7 days of episodic log entries

### Definition of Done
- A new goal is submitted, and the system demonstrably uses a skill from `/skills/` to speed up execution (referenced in the task output and log)
- `/memory/weekly-summary.md` exists and contains a human-readable summary of what the system did
- Skill reuse rate ≥ 30% measured across at least 10 task runs

### Evidence Required
- `/skills/` directory with ≥ 3 files, each with a pattern, example usage, and source task id
- At least one task log showing `skill_used: <skill-slug>` in the output
- `/memory/weekly-summary.md` with real content
- `handoff.md` records milestone achieved with commit hash

---

## Milestone 3 — Eval Harness Running and Catching Failures
**Status:** NOT STARTED
**Target:** Session 9–12 (after Milestone 2 is done)

### What It Proves
The system can automatically detect when a task output is wrong, incomplete, or low quality — not just claim success. Failures are caught before they corrupt memory.

### What It Requires
- `/eval/` directory with at least 5 test cases: each is a goal + expected output + pass/fail criteria
- An eval runner: `/eval/run_eval.py` that submits each test goal through the full pipeline and checks the output against criteria
- Failure modes covered: empty output, wrong file type, output below quality threshold, verification timeout
- Eval results written to `/eval/results-YYYYMMDD.jsonl`
- A regression check: if a task type that previously passed now fails, it is flagged in the session log

### Definition of Done
- Eval runner completes without crashing for all 5 test cases
- At least one test case is designed to fail (known bad input) and the eval correctly catches it
- `/eval/results-YYYYMMDD.jsonl` exists with pass/fail for each test case
- A deliberately introduced regression is detected and reported in the session log

### Evidence Required
- `/eval/` directory with ≥ 5 `.json` test case files
- `/eval/results-YYYYMMDD.jsonl` with results from a real run
- Session log showing a caught failure with task id, expected vs actual, and failure reason
- `handoff.md` records milestone achieved with commit hash

---

## Milestone 4 — Self-Improvement Loop Making One Measurable Improvement
**Status:** NOT STARTED
**Target:** Session 13–16 (after Milestone 3 is done)

### What It Proves
The system can identify a specific weakness in its own performance, propose a fix, apply it, and demonstrate that the fix made the metric better — with a before/after measurement.

### What It Requires
- A performance reporter: reads `/memory/episodic-log.jsonl` and `/eval/results-*.jsonl` and identifies the lowest-performing task type or worker
- A proposal writer: generates a concrete improvement proposal in `/improve/<date>-proposal.md`
- Human approval gate: Smrithi reviews and approves the proposal before it is applied
- The approved change is applied to `/workers/`, `/skills/`, or `/orchestrator/`
- A re-eval run confirms the metric improved

### Definition of Done
- One concrete, measurable improvement is made to the system (not just documented)
- Before metric and after metric both exist in `/eval/results-*.jsonl`
- The improvement is ≥ 10% better on the target metric (e.g., verification pass rate goes from 70% to 80%)
- The change was human-approved before application

### Evidence Required
- `/improve/<date>-proposal.md` with the identified weakness and proposed fix
- Two `/eval/results-*.jsonl` files: one before and one after the change
- The delta is explicitly calculated and written to `/improve/<date>-result.md`
- `handoff.md` records milestone achieved with commit hash and metric delta
