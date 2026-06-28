# /rules — Policy Rules, Trust Rules, Approval Gates

## Purpose
Rules are the policy layer of the agent OS. They define what the system is allowed to do without asking, what always requires human approval, and how trust is assigned to different actions. Rules are applied by the orchestrator before dispatching any task to a worker.

## What Goes Here
- Policy rule files (one per domain or concern)
- Trust configuration: which operations are safe, which are risky, which are forbidden
- Approval gate definitions: what triggers a pause-and-ask-human event
- Rule violation handlers: what happens when a rule is broken

## Current Rule Files

### `rules/safety.md` — Safety and Approval Gates
Defines which actions auto-proceed and which require human approval. Derived from `/system/implementation-contract.md` Safety Posture section.

### `rules/ai-calls.md` — AI API Usage Rules
Rules for all Gemini API calls: always use exponential backoff, always validate response schema, always define a fallback. (Derived from FAIL-001 and FAIL-003.)

### `rules/shell.md` — Shell Execution Rules
Rules for all PowerShell commands: display command to human before running, log output to session log, never run commands that delete files or modify system state without approval.

### `rules/memory.md` — Memory Write Rules
Rules for writing to memory: episodic log is append-only, learnings must be concrete and non-duplicate, skills must come from verified tasks only.

### `rules/git.md` — Git Operation Rules
Rules for all git operations: never force-push, never push without explicit approval, always commit session files before ending a session.

## What Does NOT Go Here
- Worker logic — that lives in `/workers/`
- System documentation — that lives in `/docs/`
- Eval test cases — those live in `/evals/`

## Current State
v1: Rule files are stubs. Actual rule content is derived from the implementation contract and failure log. Full rule files built in Session 3+.

## Rules About Rules
1. Rules are versioned — every rule file must have a version header and change log at the bottom
2. Rules cannot be overridden by an agent without recording the override in `/system/FAILURE.md`
3. New rules must be added to the index in this README before they take effect
4. Rules that become obsolete must be marked `[DEPRECATED]` but never deleted
