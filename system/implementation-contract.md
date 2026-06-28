# Implementation Contract
**System:** my-agent-os
**Owner:** Smrithi P
**Runtime:** Google Antigravity IDE — Claude Sonnet 4.6
**Version:** v1 (Session 1 baseline)
**Date:** 2026-06-28

---

## Mission

Build a personal agentic operating system that acts as Smrithi's intelligent second brain and task executor. The system accepts a goal in plain language, autonomously decomposes it into concrete sub-tasks, routes each task to the right worker (code generation, web search, file writing, shell execution), executes with minimal interruption, verifies each result is actually complete, records what it learned into persistent memory, and shows a clear activity log so Smrithi always knows what the system did and why. The mission is to multiply Smrithi's output as a developer and researcher — not to replace judgment, but to eliminate friction between idea and execution.

---

## Runtime Profile

**What this runtime CAN do:**
- Read and write every file in the repo (full filesystem access)
- Execute PowerShell commands on Smrithi's Windows machine (with approval gate)
- Search the web and read URLs without a browser
- Control a browser sub-agent for interactive web tasks
- Call external APIs (Gemini, Supabase, Firebase, etc.) via code it writes and executes
- Spawn sub-agents for parallel browser tasks
- Generate images and UI mockups
- Write, edit, and refactor code in any language in Smrithi's stack
- Persist all state to files in this repo (the filesystem IS the database for v1)

**What this runtime CANNOT do (hard limits):**
- Run always-on daemons between sessions — every session starts fresh
- Listen for inbound webhooks or events without a manually started server
- Control native desktop GUI applications (no mouse/keyboard automation)
- Execute across multiple machines
- Store secrets securely in a vault — secrets live in `.env.local` only
- Schedule tasks automatically without user or OS trigger

**Mitigation:** All missing capabilities are emulated via file checkpoints, polling, and documented manual steps. The repo is the single source of truth.

---

## First Milestone

**Accept a goal, decompose into tasks, route to worker, execute, verify result, record memory, show activity to human, learn one thing from the run.**

Concrete for Smrithi's context:

1. **Accept a goal** — Smrithi types a goal into `/inbox/goal.md` (e.g., "Scaffold a new Next.js project with Supabase auth and Gemini API integration")
2. **Decompose into tasks** — The orchestrator agent reads `goal.md`, produces a structured task list in `/tasks/active/YYYYMMDD-HHMMSS-<slug>.json` with fields: `id`, `description`, `worker_type`, `inputs`, `expected_output`, `status`
3. **Route to worker** — The router reads `worker_type` and dispatches: `code_writer`, `shell_runner`, `web_searcher`, `file_editor`, or `browser_agent`
4. **Execute** — Each worker runs and writes its raw output to `/outputs/<task-id>/result.md`
5. **Verify result** — A verifier agent checks the output against `expected_output`; writes `verified: true/false` and a short reason to `/outputs/<task-id>/verification.md`
6. **Record memory** — If verified, a memory writer appends a structured entry to `/memory/episodic-log.jsonl` with task id, goal, outcome, timestamp, and one extracted learning
7. **Show activity to human** — A summary is written to `/logs/session-YYYYMMDD.md` showing every step taken, every file touched, and the verification result; human reads this to audit the run
8. **Learn one thing from the run** — The system writes one concrete lesson to `/memory/learnings.md` (e.g., "Supabase auth scaffold requires `@supabase/ssr` not `@supabase/auth-helpers-nextjs` in Next.js 14+")

---

## Non-Goals for v1

The following are explicitly out of scope for v1 and will not be built:

- **No browser automation for arbitrary web tasks** — `browser_subagent` is used only for specific, scoped tasks (reading a page, taking a screenshot); no general "browse the web autonomously" loop
- **No multi-machine orchestration** — everything runs on Smrithi's single Windows laptop
- **No enterprise or team features** — no multi-user auth, no role-based access, no shared workspaces
- **No science or research workflows** — no paper ingestion, no dataset pipelines, no experiment tracking
- **No company operations automation** — no email sending, no calendar management, no CRM integration
- **No production deployment automation** — v1 does not push to Vercel or GCP autonomously; that requires explicit human approval
- **No self-modifying core agent code** — the self-improvement loop in v1 only modifies `/memory/` and `/skills/`; it does not rewrite the orchestrator
- **No real-time streaming UI** — the activity log is a file, not a live dashboard; a dashboard is Milestone 2+

---

## Constraints

- **Token limits:** Antigravity sessions have context limits; all persistent state must be written to files, not held in chat
- **Single machine:** All execution is on Smrithi's Windows machine; no cloud VMs for v1
- **Single user:** No auth layer, no multi-tenancy; the system trusts Smrithi completely
- **Antigravity runtime:** Every session requires a fresh context load; handoff.md and status.md are the continuity mechanism
- **Approval gate:** Any shell command with side effects (installs, network calls, file deletes) requires Smrithi's explicit approval before execution
- **Secrets:** API keys (Gemini, Supabase, Firebase) are in `.env.local`; this file is gitignored; Smrithi loads it manually per session
- **No always-on process:** The system is session-driven; it does not run between sessions without manual trigger

---

## Safety Posture

**Requires human approval before acting:**
- Any `npm install` or `pip install`
- Any git push or remote operation
- Any file delete or destructive overwrite
- Any external API call that costs money (Gemini, OpenAI, etc.) beyond the first call in a session
- Any write to a path outside this repo
- Any shell command that modifies system state (env vars, registry, PATH)
- Any deployment to Vercel, GCP Cloud Run, or any production environment

**Can auto-proceed without approval:**
- Reading any file in the repo
- Writing new files to `/tasks/`, `/outputs/`, `/memory/`, `/logs/`, `/inbox/`
- Running git status, git log, git diff (read-only git)
- Web searches and URL reads
- Generating code that has not yet been executed
- Writing to `/system/` documentation files
- Running test suites (read-only side effects)

---

## Proof-of-Progress Metrics

These are the measurable signals that confirm the system is working, not just claimed to work:

1. **Goal-to-task decomposition rate:** ≥ 90% of goals submitted to `/inbox/goal.md` produce a valid, structured task list in `/tasks/active/` within one session — measured by counting files created vs goals submitted
2. **Task execution success rate:** ≥ 80% of tasks in `/tasks/active/` reach `status: completed` with a corresponding file in `/outputs/<task-id>/result.md`
3. **Verification pass rate:** ≥ 75% of completed tasks receive `verified: true` in `/outputs/<task-id>/verification.md` — tracked in `/memory/episodic-log.jsonl`
4. **Memory record completeness:** Every verified task produces exactly one entry in `/memory/episodic-log.jsonl` — zero gaps, tracked by counting entries vs verified tasks
5. **Session log coverage:** Every session produces a `/logs/session-YYYYMMDD.md` file listing every file touched, every command run, and the final verification status — measured by existence of log file at session end
6. **Learning accumulation:** `/memory/learnings.md` grows by at least one concrete, non-trivial entry per session — measured by line count delta per session
7. **Handoff continuity:** Every session ends with an updated `handoff.md`; next session can read it and resume without context loss — measured by whether Session N+1 can locate its start point from `handoff.md` alone
8. **Skill reuse rate:** By Milestone 2, at least 30% of task executions reuse a pattern from `/skills/` rather than generating from scratch — measured by grep for skill references in task outputs

---

## Verification Strategy

No task is marked done without evidence. The verification chain is:

1. **Definition of done written before execution** — every task in `/tasks/active/<id>.json` must have `expected_output` populated before the task runs
2. **Output file must exist** — `/outputs/<task-id>/result.md` must be created by the worker; a task with no output file is `status: failed`
3. **Verifier reads expected vs actual** — a separate verification step (not the same agent that did the work) compares `expected_output` to the actual content of `result.md`
4. **Verification result is written to a file** — `/outputs/<task-id>/verification.md` with fields: `verified`, `reason`, `timestamp`; this file is the canonical truth
5. **Episodic log entry only on verified:true** — unverified tasks are logged as failed, not silently dropped
6. **Session summary includes verification counts** — `/logs/session-YYYYMMDD.md` always shows: tasks attempted, tasks completed, tasks verified, tasks failed
7. **Handoff includes open tasks** — any task not verified by end of session is listed in `handoff.md` under `blocked` or `carry-forward` so Session N+1 can pick it up
