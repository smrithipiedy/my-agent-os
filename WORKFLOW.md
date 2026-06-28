# Workflow
**System:** my-agent-os
**Version:** v1 — Guided Autonomy Mode
**Updated:** Session 2, 2026-06-28

Every phase of this workflow MUST leave a file artifact trail. If a phase produces no file, it did not happen.

---

## Overview: The Eight-Phase Pipeline

```
Goal Intake → Task Graph → Human Approval → Worker Claims → Execute → Verifier Checks → Evidence Saved → Memory Updated → Session Recorded
```

---

## Phase 1: Goal Intake

**Trigger:** Smrithi writes a goal in free-form plain text.

**Action:**
1. Smrithi opens `/inbox/goal.md` and writes her goal
2. The goal is saved with a timestamp comment at the top
3. Smrithi invokes the orchestrator (runs `/orchestrator/run.py`)

**Files written this phase:**
- `/inbox/goal.md` — the raw goal text (written by Smrithi)

**File written by orchestrator:**
- `/inbox/goal-<timestamp>.archive.md` — a copy of the goal preserved before the inbox is cleared for the next run

**Failure at this phase:**
- `/inbox/goal.md` is empty → orchestrator exits with error: `ERROR: inbox/goal.md is empty. Write a goal first.`
- No file written. Session log records: `goal_intake: FAILED — empty inbox`

---

## Phase 2: Task Graph Construction

**Trigger:** Orchestrator reads `/inbox/goal.md` and calls Gemini API.

**Action:**
1. Orchestrator sends goal text to Gemini with a decomposition prompt
2. Gemini returns a structured list of 2–7 tasks
3. Orchestrator validates each task has required fields: `description`, `worker_type`, `expected_output`
4. Orchestrator assigns a unique `task_id` to each task (format: `<YYYYMMDD-HHMMSS>-<slug>`)
5. Orchestrator assigns a `goal_id` (format: `goal-<YYYYMMDD-HHMMSS>`)
6. Each task is written as an individual JSON file

**Files written this phase:**
- `/tasks/active/<goal-id>-task-<N>.json` — one file per task, all fields populated
- `/logs/session-YYYYMMDD.md` — first entry written: `[HH:MM:SS] TASK GRAPH: <N> tasks decomposed from goal_id=<goal-id>`

**Failure at this phase:**
- Gemini API call fails → orchestrator writes `/tasks/failed/<goal-id>-decomposition-error.json` with error details; exits
- Gemini returns < 2 tasks or > 7 tasks → orchestrator rejects and logs: `DECOMPOSITION: rejected — <N> tasks outside valid range [2,7]`
- Any task missing `expected_output` → orchestrator rejects that task and logs a warning; does not proceed to approval

---

## Phase 3: Human Approval (Guided Autonomy Gate)

**Trigger:** Task graph is complete. Orchestrator presents it to Smrithi before any execution.

**Action:**
1. Orchestrator prints a formatted summary of all tasks to the terminal:
   - Task ID, description, worker_type, expected_output
2. Smrithi reviews and types: `approve`, `reject`, or `approve <task-ids>` (to approve a subset)
3. Orchestrator records the decision

**Files written this phase:**
- `/tasks/active/<task-id>.json` — `status` field updated from `pending_approval` to `approved` or `rejected`
- `/logs/session-YYYYMMDD.md` — entry: `[HH:MM:SS] APPROVAL: <N> tasks approved, <M> tasks rejected by human`

**Failure at this phase:**
- Smrithi rejects all tasks → orchestrator exits cleanly; logs: `APPROVAL: all tasks rejected — session ended`
- No response within session → tasks remain `pending_approval`; carried forward in next session's handoff

---

## Phase 4: Worker Claims Task

**Trigger:** A task has `status: approved`. Orchestrator dispatches it to the correct worker.

**Action:**
1. Orchestrator reads `worker_type` from task JSON
2. Looks up handler in `/workers/registry.json`
3. Instantiates the worker and passes the full task dict
4. Worker updates `status` field to `in_progress` in the task JSON

**Files written this phase:**
- `/tasks/active/<task-id>.json` — `status` updated to `in_progress`, `started_at` field populated
- `/logs/session-YYYYMMDD.md` — entry: `[HH:MM:SS] WORKER CLAIM: task=<id> claimed by worker=<worker_type>`

**Failure at this phase:**
- `worker_type` not found in registry → task immediately set to `status: failed`, `escalation_reason: unknown worker_type`; logged
- Worker crashes on init → task set to `status: failed`, error written to `/outputs/<task-id>/error.md`

---

## Phase 5: Execution

**Trigger:** Worker has claimed the task and begins processing.

**Action:**
1. Worker executes its logic (calls Gemini, runs shell command, searches web, edits file, etc.)
2. Worker writes its output to `/outputs/<task-id>/result.md`
3. Worker updates task JSON: `status: completed`, `completed_at`, `tokens_used`, `attempts`

**Files written this phase:**
- `/outputs/<task-id>/result.md` — the worker's actual output (required; task fails without this)
- `/tasks/active/<task-id>.json` — `status`, `completed_at`, `tokens_used`, `attempts` updated
- `/logs/session-YYYYMMDD.md` — entry: `[HH:MM:SS] EXECUTE: task=<id> worker=<type> status=completed tokens=<N>`

**Failure at this phase:**
- Worker produces no `result.md` → task set to `status: failed`; log entry written
- Worker raises an exception → error written to `/outputs/<task-id>/error.md`; task set to `status: failed`; if `attempts < 3`, task is retried once automatically; second failure escalates to human
- Shell command requires approval → worker pauses, prints request to terminal, waits; if no approval, task set to `status: waiting_approval`

---

## Phase 6: Verifier Checks Output

**Trigger:** `result.md` exists. A separate verification step runs — never the same worker that ran the task.

**Action:**
1. Verifier reads `expected_output` from task JSON
2. Verifier reads actual content of `/outputs/<task-id>/result.md`
3. Verifier calls Gemini with a structured prompt: "Does the actual output satisfy this expected output? Answer yes/no with reason."
4. Verifier writes the result

**Files written this phase:**
- `/outputs/<task-id>/verification.md` — fields: `verified` (bool), `reason` (string), `timestamp` (ISO 8601), `verifier` (string, default: `gemini-verification-v1`)
- `/tasks/active/<task-id>.json` — `verified` field updated
- `/logs/session-YYYYMMDD.md` — entry: `[HH:MM:SS] VERIFY: task=<id> verified=<true/false> reason=<truncated>`

**Failure at this phase:**
- Verifier cannot reach Gemini API → `verification.md` written with `verified: false`, `reason: verifier_unavailable`; task flagged for manual review
- `result.md` is empty → `verified: false`, `reason: empty_output`

---

## Phase 7: Evidence Saved

**Trigger:** `verification.md` exists. Evidence archiving runs.

**Action:**
1. System confirms all required files exist for this task: task JSON, result.md, verification.md
2. If `verified: true`: writes episodic log entry to `/memory/episodic-log.jsonl`
3. If `verified: false`: writes failure log entry to `/memory/episodic-log.jsonl` with `outcome: failed`
4. Extracts one learning from the task output and appends to `/memory/learnings.md`

**Files written this phase:**
- `/memory/episodic-log.jsonl` — one new JSONL line appended (always, whether verified or not)
- `/memory/learnings.md` — one new entry appended (only for `verified: true`)

**Failure at this phase:**
- Memory files cannot be written (disk full, permissions) → error logged to session log; task marked `memory_write_failed`; human notified

---

## Phase 8: Session Recorded

**Trigger:** All tasks in the current goal have been processed (completed, failed, or waiting_approval).

**Action:**
1. Session log finalised with summary section
2. `status.md` updated
3. `handoff.md` updated with carry-forward tasks and open questions
4. `system/momentum.md` updated
5. All files committed to git

**Files written this phase:**
- `/logs/session-YYYYMMDD.md` — summary section appended: `tasks_attempted`, `tasks_completed`, `tasks_verified`, `tasks_failed`
- `status.md` — updated fields: `current_session`, `last_action`, `next_action`, `system_health`
- `handoff.md` — updated with carry-forward tasks, blockers, open questions for next session
- `system/momentum.md` — queues updated: NOW cleared, NEXT populated from carry-forward
- Git commit: `Session N: <goal-slug> — <N> tasks, <M> verified`

**Failure at this phase:**
- Git commit fails → error logged; human notified; files are still on disk (no data loss)

---

## Failure Handling Summary

| Phase | Failure Mode | Auto-Recovery | Escalate to Human |
|---|---|---|---|
| Goal Intake | Empty inbox | No — exits cleanly | No |
| Task Graph | Gemini API failure | Retry once | Yes if retry fails |
| Task Graph | Invalid decomposition | No — reject and exit | Yes |
| Human Approval | All tasks rejected | No — exits cleanly | No |
| Worker Claims | Unknown worker_type | No — fail task immediately | Yes |
| Execution | No result.md | Retry once (if attempts < 3) | Yes if retry fails |
| Execution | Shell needs approval | Pause and wait | Yes — always |
| Verification | Verifier unavailable | Mark for manual review | Yes |
| Evidence Save | Memory write fails | No | Yes |
| Session Record | Git commit fails | No | Yes — data still safe |

---

## File Artifact Trail (Complete Map)

Every goal run produces this minimum set of files:

```
/inbox/
  goal.md                              ← Smrithi writes this
  goal-<timestamp>.archive.md          ← Orchestrator preserves it

/tasks/active/
  <goal-id>-task-1.json               ← One file per task
  <goal-id>-task-2.json
  ...

/outputs/
  <task-id>/
    result.md                          ← Worker output (required)
    verification.md                    ← Verifier result (required)
    error.md                           ← Only on failure

/memory/
  episodic-log.jsonl                   ← One JSONL line per task
  learnings.md                         ← One entry per verified task

/logs/
  session-YYYYMMDD.md                  ← Full audit trail

/                                      ← Root continuity files
  status.md
  handoff.md
```

If any of these files are missing at end of session, the system did not complete that phase. No exceptions.
