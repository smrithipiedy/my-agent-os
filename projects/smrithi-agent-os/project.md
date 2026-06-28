# Project Charter: Smrithi's Agent OS

## Project Name
Smrithi Agent OS — Personal Agentic Operating System (v1)

## Owner
Smrithi P — final year CSE, full-stack software dev, open source contributor, aspiring agentic AI engineer

## Goal
Build a personal agentic operating system that acts as Smrithi's intelligent second brain and autonomous task executor. The system accepts goals in plain English, decomposes them into concrete tasks using the Gemini API, routes each task to the appropriate worker (code writer, shell runner, web searcher, file editor, browser agent), executes with human approval for risky operations, verifies every result, records learnings into persistent memory, and shows a full audit trail so Smrithi can see exactly what happened. The mission is to multiply Smrithi's output as a developer by eliminating the friction between idea and execution — not to replace judgment, but to automate the repeatable parts of software development and research.

## Success Criteria
- [ ] **M1:** Full 8-step pipeline works end-to-end: goal → decompose → approve → execute → verify → memory → log → learn — with all evidence files present in the repo
- [ ] **M2:** Memory and skill system works: system reuses existing patterns for ≥ 30% of tasks; `episodic-log.jsonl` has ≥ 10 real entries; `/skills/` has ≥ 3 extracted skills
- [ ] **M3:** Eval harness catches regressions: ≥ 5 eval cases run automatically; at least one deliberate failure is caught; results written to `/evals/results-*.jsonl`
- [ ] **M4:** Self-improvement loop makes one measurable improvement: before/after metrics both exist; improvement ≥ 10% on target metric; change was human-approved
- [ ] Verification pass rate ≥ 80% sustained across ≥ 10 real task runs
- [ ] Session handoff continuity: every session produces a `handoff.md` that the next session can start from without reading chat history

## Non-Goals
- No enterprise features, no multi-user support, no role-based access
- No production deployment automation without explicit human approval
- No browser automation beyond specific scoped tasks
- No multi-machine orchestration (single Windows laptop only)
- No real-time streaming dashboard in v1 (file logs only)
- No self-modifying orchestrator code in the self-improvement loop

## Timeline
**Start:** Session 1, 2026-06-28
**Target M1 completion:** Session 4
**Target M2 completion:** Session 8
**Target M3 completion:** Session 12
**Target M4 completion:** Session 16
**Current phase:** Session 2 — Foundational Artifacts (Build Order Step 3)

## Dependencies
- `GEMINI_API_KEY` in `.env.local` ✅ (confirmed by Smrithi, Session 2)
- Python installed on Windows machine (needs verification in Session 3)
- `google-generativeai` Python package (to be installed Session 3)
- Git initialized ✅ (done Session 1)
- `.gitignore` configured ✅ (done between Session 1 and 2 by Smrithi)

## Risk Level
**Risk:** Medium
**Highest risk:** The Gemini API is the critical path — every major feature depends on it. Quota exhaustion or API changes could stall multiple sessions. Mitigation: exponential backoff, model fallback chain, offline fallback for verifiable tasks.

## Project ID
proj-20260628-smrithi-agent-os
