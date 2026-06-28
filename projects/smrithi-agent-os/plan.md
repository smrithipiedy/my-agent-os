# Project Plan: Smrithi Agent OS

## Milestones

| # | Milestone | Target Session | Status | Definition of Done |
|---|---|---|---|---|
| 1 | Full Loop: Goal→Result→Memory→Human | Session 4 | NOT STARTED | All 8 steps produce evidence files for one real goal run |
| 2 | Memory and Skill System | Session 8 | NOT STARTED | ≥10 episodic entries, ≥3 skills, skill reuse rate ≥30% |
| 3 | Eval Harness Running | Session 12 | NOT STARTED | ≥5 eval cases run, regressions caught, results in JSONL |
| 4 | Self-Improvement Loop | Session 16 | NOT STARTED | One measurable improvement with before/after metrics |

## Workstream Breakdown

### Workstream 1: Core Infrastructure (Sessions 1–4)
**Owner:** Smrithi P + Antigravity agent
**Sessions:** 1–4
**Status:** IN PROGRESS (Session 2 of 4)
**Description:** Build the foundational structure — system docs, folder hierarchy, orchestrator, workers, first end-to-end run
**Tasks:**
- [x] Session 1: Capability matrix, implementation contract, milestones, momentum, status, handoff
- [x] Session 2: REQUIREMENTS.md, WORKFLOW.md, system contracts, knowledge base, memory index, all folder READMEs, project template, this project folder
- [ ] Session 3: Orchestrator (`/orchestrator/run.py`), inbox system, task schema, worker registry, first worker (`code_writer.py`), setup script, env validation
- [ ] Session 4: First full end-to-end run, verification chain, memory writes, session log, Milestone 1 evidence

### Workstream 2: Memory and Skills (Sessions 5–8)
**Owner:** Smrithi P + Antigravity agent
**Sessions:** 5–8
**Status:** NOT STARTED
**Description:** Build the memory persistence layer, skill extraction, and SQLite index
**Tasks:**
- [ ] Session 5: Episodic log writer, learnings extractor, memory reader
- [ ] Session 6: Skill extractor, skills registry, SQLite index builder
- [ ] Session 7: Weekly summary generator, memory retrieval in orchestrator
- [ ] Session 8: Skill reuse in workers, metrics check (≥30% reuse), Milestone 2 evidence

### Workstream 3: Eval Harness (Sessions 9–12)
**Owner:** Smrithi P + Antigravity agent
**Sessions:** 9–12
**Status:** NOT STARTED
**Description:** Build automated quality gates, regression detection, and eval runner
**Tasks:**
- [ ] Session 9: Define ≥5 eval cases; build eval runner skeleton
- [ ] Session 10: Run evals, fix failures, establish baseline
- [ ] Session 11: Add regression detection; build schema validators
- [ ] Session 12: Milestone 3 evidence; eval results in JSONL

### Workstream 4: Self-Improvement Loop (Sessions 13–16)
**Owner:** Smrithi P + Antigravity agent
**Sessions:** 13–16
**Status:** NOT STARTED
**Description:** Build performance reporter, proposal writer, and measurable improvement cycle
**Tasks:**
- [ ] Session 13: Performance reporter — reads episodic log, identifies lowest-performing area
- [ ] Session 14: Proposal writer — generates `/improve/<date>-proposal.md`
- [ ] Session 15: Apply approved change, run before/after evals
- [ ] Session 16: Milestone 4 evidence; improvement ≥10% documented

## Current Focus
**Session 2:** Completing Build Order Step 3 — all foundational artifacts and canonical folder structure. No code yet. The goal is a complete, navigable repo structure that any future AI agent can orient from.

## Decisions Pending
| Decision | Needed By | Impact If Deferred |
|---|---|---|
| Confirm Python version (3.10? 3.11? 3.12?) | Session 3 | Affects f-string features and `tomllib` stdlib availability |
| Hub implementation language: Python (FastAPI) or Node.js (Express)? | Session 5 | Affects workstream 2 start |

## Carry-Forward from Last Session
| Task ID | Status | What Remains |
|---|---|---|
| Session 1 bootstrap | ✅ COMPLETE | N/A — all Session 1 files committed |
