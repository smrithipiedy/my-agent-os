# Requirements
**System:** my-agent-os
**Owner:** Smrithi P
**Runtime:** Google Antigravity IDE — Claude Sonnet 4.6 (Thinking)
**Version:** v1
**Source:** Derived from `/system/implementation-contract.md`
**Updated:** Session 2, 2026-06-28

---

## Functional Requirements
*What the system must do*

### FR-1: Goal Intake
- The system MUST accept a goal written in free-form plain text placed in `/inbox/goal.md`
- The system MUST support one active goal at a time in v1
- The system MUST NOT require structured YAML/JSON input from Smrithi — plain English is sufficient
- The system MUST preserve the original goal text verbatim in all task records and memory entries

### FR-2: Task Decomposition
- The system MUST decompose every accepted goal into 2–7 concrete sub-tasks using the Gemini API
- Each task MUST be written as a JSON file to `/tasks/active/` with the full schema defined in `/projects/_template/tasks.md`
- Every task JSON MUST contain a non-empty `expected_output` field before execution begins
- The system MUST present the decomposed task list to Smrithi for approval before any task executes (guided autonomy mode)

### FR-3: Worker Routing
- The system MUST route each task to exactly one worker type based on the task's `worker_type` field
- Valid worker types for v1: `code_writer`, `shell_runner`, `web_searcher`, `file_editor`, `browser_agent`
- The worker registry MUST be defined in `/workers/registry.json`
- An unknown `worker_type` MUST cause the task to fail with a clear error — no silent fallthrough

### FR-4: Task Execution
- Each worker MUST write its output to `/outputs/<task-id>/result.md`
- A task with no `result.md` at end of execution MUST be marked `status: failed`
- Workers MUST NOT modify files outside the repo directory without explicit human approval
- Shell commands with side effects (installs, deletes, writes outside repo) MUST pause and request approval

### FR-5: Verification
- Every completed task MUST be verified by a separate check — not by the worker that ran it
- Verification MUST compare actual `result.md` content against the task's `expected_output`
- Verification result MUST be written to `/outputs/<task-id>/verification.md` with: `verified` (bool), `reason` (string), `timestamp` (ISO 8601)
- Only tasks with `verified: true` are recorded in memory; failed tasks are logged separately

### FR-6: Memory Recording
- Every task with `verified: true` MUST produce exactly one entry in `/memory/episodic-log.jsonl`
- The entry MUST contain: `task_id`, `goal_id`, `goal_text`, `worker_type`, `outcome_summary`, `learning`, `timestamp`
- Every session MUST append at least one entry to `/memory/learnings.md`

### FR-7: Human Activity Log
- Every session MUST produce a file at `/logs/session-YYYYMMDD.md`
- The log MUST list every file touched, every command run, every task attempted/completed/verified/failed
- The log MUST include timestamps for each action
- The log MUST include a summary section: tasks_attempted, tasks_completed, tasks_verified, tasks_failed

### FR-8: Skill Reuse (Milestone 2 target)
- The skills registry at `/skills/` MUST be searched before generating new code for a task
- When a skill is used, the task output MUST reference the skill slug (`skill_used: <slug>`)
- Skill reuse rate MUST be ≥ 30% across 10+ task runs by end of Milestone 2

---

## Constraints
*What the system must never do*

### CN-1: Safety Constraints
- The system MUST NEVER execute a shell command with side effects without Smrithi's explicit approval
- The system MUST NEVER commit secrets, API keys, or `.env.local` contents to git
- The system MUST NEVER push to a remote git repository without explicit approval
- The system MUST NEVER deploy to any production environment (Vercel, GCP, Firebase) without explicit approval
- The system MUST NEVER delete files without explicit approval
- The system MUST NEVER mark a task `verified: true` without a corresponding `verification.md` file existing on disk

### CN-2: Scope Constraints (v1 Non-Goals)
- The system MUST NOT perform browser automation beyond specific scoped tasks
- The system MUST NOT operate across multiple machines
- The system MUST NOT include multi-user authentication or role-based access
- The system MUST NOT manage email, calendar, or CRM systems
- The system MUST NOT rewrite the orchestrator or core agent code as part of the self-improvement loop
- The system MUST NOT render a real-time dashboard in v1 (file logs only)

### CN-3: Data Constraints
- All persistent state MUST be stored in files within this repo
- SQLite (via `/.system/agent.db`) may be used for indexed queries over memory — but it is a cache, not the source of truth; files are canonical
- Memory files MUST NOT be manually edited by the agent without writing a corresponding log entry

---

## First Milestone Requirements
*Concretely what Milestone 1 requires — all eight steps must work end-to-end*

| Step | Requirement | Evidence File |
|---|---|---|
| 1. Accept goal | `/inbox/goal.md` contains a real goal | `/inbox/goal.md` |
| 2. Decompose | Task JSON files exist in `/tasks/active/` | `/tasks/active/<id>.json` |
| 3. Route | Worker selected from registry, task dispatched | Task JSON `worker_type` field |
| 4. Execute | Worker output file written | `/outputs/<id>/result.md` |
| 5. Verify | Verification file written with `verified: true` | `/outputs/<id>/verification.md` |
| 6. Record memory | Episodic log entry written | `/memory/episodic-log.jsonl` |
| 7. Show human | Session log written | `/logs/session-YYYYMMDD.md` |
| 8. Learn | Learning entry appended | `/memory/learnings.md` |

**Milestone 1 is NOT done until all 8 evidence files exist in the repo for the same goal run.**

---

## How Requirements Are Verified

### Automated checks (run via `/evals/` harness, Milestone 3):
- `check_task_schema.py` — validates every file in `/tasks/active/` against the JSON schema
- `check_verification_chain.py` — for every `result.md`, confirms a `verification.md` exists and is non-empty
- `check_memory_completeness.py` — counts episodic-log entries vs verified tasks; flags gaps
- `check_log_coverage.py` — confirms session log exists and contains required sections

### Manual checks (run by Smrithi after every session until Milestone 3):
1. Open the session log at `/logs/session-YYYYMMDD.md` and confirm all 7 sections are present
2. Open `/memory/episodic-log.jsonl` and confirm new entries were added
3. Open `/memory/learnings.md` and confirm at least one new entry since last session
4. Check `handoff.md` is dated today and references unresolved tasks

### Proof-of-Progress Metrics (from implementation contract):
1. Goal-to-task decomposition rate ≥ 90%
2. Task execution success rate ≥ 80%
3. Verification pass rate ≥ 80%
4. Memory record completeness: zero gaps
5. Session log coverage: log exists every session
6. Learning accumulation: ≥ 1 new entry per session
7. Handoff continuity: next session can start from handoff.md alone
8. Skill reuse rate ≥ 30% by Milestone 2
