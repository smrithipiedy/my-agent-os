# System Contracts
**System:** my-agent-os
**Version:** v1
**Source:** Derived from `/system/implementation-contract.md` Verification Strategy section
**Updated:** Session 2, 2026-06-28

These contracts are non-negotiable. Every agent, worker, and session must satisfy them. Violations must be logged in `/system/FAILURE.md`.

---

## Contract 1: Definition of Done

A task is done if and only if ALL of the following are true:

```
✓ /tasks/active/<task-id>.json  status == "completed"
✓ /outputs/<task-id>/result.md  exists and is non-empty
✓ /outputs/<task-id>/verification.md  exists with verified == true
✓ /memory/episodic-log.jsonl  contains an entry with this task_id
✓ /logs/session-YYYYMMDD.md  contains a log line for this task
```

**If any one of these five conditions is false, the task is NOT done.**

Do not mark a task done in a summary, in chat, or in a handoff unless all five files exist with the correct content. Saying "it's done" in text without file evidence is a protocol violation.

---

## Contract 2: Evidence Contract

What counts as proof a task is complete:

### Required Evidence Files (per task)

| Evidence File | Required Fields | What It Proves |
|---|---|---|
| `/tasks/active/<id>.json` | `status: "completed"`, `completed_at: <ISO timestamp>` | Task reached the completed state |
| `/outputs/<id>/result.md` | Non-empty content matching the task description | Worker produced real output |
| `/outputs/<id>/verification.md` | `verified: true`, `reason: <string>`, `timestamp: <ISO>` | Output was independently checked |
| `/memory/episodic-log.jsonl` | JSONL entry with `task_id`, `outcome`, `learning` | Learning was recorded |
| `/logs/session-YYYYMMDD.md` | Log line referencing this task ID | Activity was auditable |

### What Does NOT Count as Evidence

- A summary in chat saying the task is done
- An updated `handoff.md` saying "task X completed" without the five files above
- A `result.md` that exists but contains only the task description re-stated
- A `verification.md` that says `verified: true` without a `reason`
- Any file created after the session ended (backdated files are not valid evidence)

### Evidence Grading

| Grade | Condition |
|---|---|
| COMPLETE | All 5 evidence files exist with valid content |
| PARTIAL | 3–4 of 5 evidence files exist — task is in progress, not done |
| FAILED | result.md exists but verification.md says `verified: false` |
| MISSING | result.md does not exist — worker did not run or crashed |

---

## Contract 3: Handoff Contract

Every session MUST write a `handoff.md` before ending. The file is overwritten each session. It MUST contain:

### Required Sections

```markdown
## What Was Completed This Session
[List every file created or meaningfully modified, with one line explaining its purpose]

## Carry-Forward Tasks
[List every task that was started but not verified — include task_id, current status, and what remains]

## Blockers
[List every capability gap, missing credential, or decision needed before next session can proceed]

## What Session N+1 Must Do
[Numbered list of specific, actionable tasks. Each must reference the Build Order step number or file it touches]

## Open Questions
[Table with: # | question | who answers | priority | deadline]
```

### Handoff Quality Rules

- MUST be written before the git commit that ends the session
- MUST be specific — "continue building the orchestrator" is not acceptable; "complete Phase 5 of WORKFLOW.md in `/orchestrator/run.py`" is acceptable
- MUST list every task with `status: in_progress` at session end
- MUST NOT claim any task is complete if its evidence contract is not satisfied
- The next session agent MUST be able to resume work from `handoff.md` alone, without reading chat history

---

## Contract 4: Memory Contract

After every task with `verified: true`, the system MUST write to memory:

### Memory Write Checklist

```
✓ /memory/episodic-log.jsonl  — append one JSONL line
✓ /memory/learnings.md        — append one learning entry (if learning is novel)
```

### Episodic Log Entry Schema

```json
{
  "task_id": "YYYYMMDD-HHMMSS-slug",
  "goal_id": "goal-YYYYMMDD-HHMMSS",
  "session": "Session N",
  "timestamp": "ISO 8601",
  "worker_type": "code_writer|shell_runner|web_searcher|file_editor|browser_agent",
  "goal_text": "Free-form goal Smrithi wrote",
  "outcome": "verified|failed",
  "outcome_summary": "One sentence describing what the worker produced",
  "learning": "One concrete, specific lesson extracted from this task run",
  "tokens_used": 0,
  "skill_used": null
}
```

### Learning Entry Format (in `/memory/learnings.md`)

```markdown
### LEARN-<NNN>
**Date:** YYYY-MM-DD
**Session:** Session N
**Source Task:** <task_id>
**Context:** <the situation or technology this applies to>
**Learning:** <one concrete, specific, actionable insight>
**Applies To:** <worker_type or domain>
```

### Memory Contract Rules

- A task with `verified: false` MUST still get an episodic log entry — with `outcome: "failed"`
- Learning entries MUST be concrete and specific — "Gemini sometimes returns null fields" is not acceptable; "Gemini `gemini-1.5-flash` returns null for `candidates[0].content` when the prompt triggers a safety filter — check for null before accessing content" is acceptable
- Duplicate learnings MUST NOT be written — check if the same learning already exists before appending
- Memory files are append-only — never delete or edit existing entries

---

## Contract 5: Session Log Contract

Every session MUST produce `/logs/session-YYYYMMDD.md`. The file MUST contain:

### Required Sections

```markdown
# Session Log — YYYY-MM-DD
**Session:** N
**Start:** HH:MM:SS
**End:** HH:MM:SS

## Goal
[Goal text from /inbox/goal.md]

## Actions Taken
[Timestamped list of every action: files written, commands run, API calls made]

## Files Written This Session
[Complete list of file paths created or modified]

## Task Results
| Task ID | Worker | Status | Verified |
|---|---|---|---|

## Summary
- tasks_attempted: N
- tasks_completed: N
- tasks_verified: N
- tasks_failed: N
- tokens_used_total: N
- learnings_written: N
```

### Session Log Rules

- MUST be written during the session — not reconstructed from memory after
- MUST include a timestamp for every action
- MUST include every file path written (not just a count)
- MUST be committed to git as part of the session-ending commit
