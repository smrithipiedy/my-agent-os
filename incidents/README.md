# /incidents — Incident Log and Templates

## Purpose
Incidents are distinct from failures (in `/system/FAILURE.md`). Failures are technical issues during task execution. Incidents are broader events — a session that produced corrupted memory, a worker that ran out of control, a goal that caused unintended side effects. Incidents require a post-mortem, not just a bug fix.

## What Goes Here
- `_template.md` — the incident report template
- One Markdown file per incident (e.g., `INC-001-memory-corruption-2026-06-28.md`)
- `incidents-index.md` — running index of all incidents with severity and resolution status

## Incident Severity Levels

| Level | Description | Example |
|---|---|---|
| SEV-1 | System-wide data loss or corruption | Memory files overwritten; episodic log truncated |
| SEV-2 | Task pipeline broken for >1 session | Orchestrator fails to start; all tasks stuck in pending |
| SEV-3 | Single task type consistently failing | code_writer worker crashes on all Python tasks |
| SEV-4 | Minor degradation | Verification producing false negatives; one task failed unexpectedly |

## Incident Report Template

```markdown
# Incident: INC-<NNN>
**Date:** YYYY-MM-DD
**Session:** Session N
**Severity:** SEV-1 | SEV-2 | SEV-3 | SEV-4
**Status:** OPEN | INVESTIGATING | RESOLVED

## Summary
One paragraph describing what happened.

## Timeline
- HH:MM: First sign of problem
- HH:MM: Impact confirmed
- HH:MM: Mitigation applied
- HH:MM: Resolution confirmed

## Root Cause
What actually caused this — be specific.

## Impact
What did not work as a result. Which tasks failed. What memory may be incomplete.

## Resolution
What was done to fix it.

## Preventative Actions
What was changed (in rules, code, or docs) to prevent recurrence.
```

## What Does NOT Go Here
- Individual task failures — those go in `/system/FAILURE.md`
- Bug reports about specific workers — those are tracked in `/projects/<project>/tasks.md` with `risk_level: high`

## Current State
v1: No incidents yet. The directory is created to reserve the namespace.

## Rules
1. Any event with system-wide impact (SEV-1 or SEV-2) MUST have an incident report written within the same session it is discovered
2. Incident reports are never deleted — mark them RESOLVED but keep them permanently
3. Every incident must have at least one preventative action — "we'll be more careful" is not a preventative action
