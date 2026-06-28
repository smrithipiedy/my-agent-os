# Memory Index
**System:** my-agent-os
**Type:** Index document — describes all memory layer files
**Updated:** Session 2, 2026-06-28

This file is the map of the memory system. It describes what files exist, their type, when they are updated, and how to retrieve from them. Actual memory files live in `/memory/`. The actual memory layer implementation is built in Session 6.

---

## Memory Type Definitions

| Type | Description | Analogy |
|---|---|---|
| **Episodic** | Records of specific events that happened (what, when, outcome) | A diary |
| **Semantic** | Structured facts and knowledge that do not belong to a specific event | An encyclopedia |
| **Procedural** | How to do things — step-by-step patterns and skills | A recipe book |
| **Preference** | Smrithi's stated preferences, working style, and decisions | A profile |
| **Temporal** | Time-bounded records — session logs, daily summaries | A calendar |
| **Hot** | Accessed every session; kept in fast-access files | RAM |
| **Warm** | Accessed weekly or when relevant; queryable via SQLite index | SSD |
| **Cold** | Rarely accessed; full history; append-only JSONL | Archive |

---

## Memory File Registry

### `/memory/episodic-log.jsonl`
- **Type:** Episodic, Cold
- **Temperature:** Cold (full history, append-only)
- **Format:** JSONL — one JSON object per line
- **Updated:** After every task completes (verified or failed)
- **Contains:** task_id, goal_id, session, timestamp, worker_type, goal_text, outcome, outcome_summary, learning, tokens_used, skill_used
- **How to retrieve:** `grep "task_id" /memory/episodic-log.jsonl` for exact lookups; or query SQLite index at `/.system/agent.db` table `episodic_events` for filtered queries
- **Built in:** Session 6 (implementation); first entries written in Session 3 (Milestone 1 test run)
- **Status:** FILE EXISTS (empty), schema defined in `/system/contracts.md`

---

### `/memory/learnings.md`
- **Type:** Semantic, Warm
- **Temperature:** Warm (checked at start of each session for relevant context)
- **Format:** Markdown — structured entries with headers (LEARN-NNN format)
- **Updated:** After every task with `verified: true`; at least once per session
- **Contains:** Concrete, actionable technical lessons extracted from completed tasks
- **How to retrieve:** Read the full file (it should stay under 500 lines; split if it grows beyond that); grep for keywords; no SQLite index needed for v1
- **Built in:** Session 3 (first entries written during Milestone 1 run)
- **Status:** NOT YET CREATED (created when first entry is written)

---

### `/memory/skills-index.md`
- **Type:** Procedural, Warm
- **Temperature:** Warm (checked before every code generation task)
- **Format:** Markdown — index of skill slugs with one-line descriptions and file paths
- **Updated:** When a new skill is extracted and written to `/skills/`
- **Contains:** List of all available skill files with: slug, description, worker_type, source_task_id, last_used date
- **How to retrieve:** Scan the index for relevant skill slugs, then read `/skills/<slug>.md` for the full pattern
- **Built in:** Session 6 (Milestone 2 prerequisite)
- **Status:** NOT YET CREATED

---

### `/memory/preferences.md`
- **Type:** Preference, Hot
- **Temperature:** Hot (read at start of every session by the orchestrator)
- **Format:** Markdown — key-value style with sections
- **Updated:** When Smrithi states a new preference or changes an existing one
- **Contains:**
  - Working style: autonomy level (guided), preferred languages (Python, JavaScript/TypeScript)
  - Stack preferences: Next.js, Firebase, Supabase, Gemini, Vercel, GCP
  - Communication style: Smrithi prefers concise summaries; no verbose explanations unless asked
  - Risk tolerance: cautious on shell commands; permissive on file writes
  - Goal format: free-form plain text is preferred
- **How to retrieve:** Read the entire file at session start; it is small and changes infrequently
- **Built in:** Session 2 (this session) — stub created; filled in over time
- **Status:** TO BE CREATED this session

---

### `/memory/weekly-summary.md`
- **Type:** Temporal, Warm
- **Temperature:** Warm (generated weekly from episodic log)
- **Format:** Markdown — one section per week with goal summaries and key learnings
- **Updated:** Every 7 sessions, or manually triggered
- **Contains:** Summary of goals processed, tasks completed, verified tasks, top learnings, skills extracted
- **How to retrieve:** Read the file; most recent week is at the bottom
- **Built in:** Milestone 2 (Session 6+)
- **Status:** NOT YET CREATED

---

### `/.system/agent.db`
- **Type:** Semantic index (cache), Hot
- **Temperature:** Hot (rebuilt at session start from JSONL if stale)
- **Format:** SQLite database
- **Updated:** Synced from `episodic-log.jsonl` at start of each session
- **Contains:** Tables: `episodic_events`, `skills`, `preferences`
- **How to retrieve:** Standard SQL queries via `sqlite3` Python module
- **Gitignored:** Yes — regeneratable; never committed
- **Built in:** Session 6 (Milestone 2 prerequisite)
- **Status:** NOT YET CREATED (gitignore entry already in place)

---

### `/logs/session-YYYYMMDD.md`
- **Type:** Temporal, Cold
- **Temperature:** Cold (created once per session; read only for audit)
- **Format:** Markdown — structured log with timestamp headers
- **Updated:** Continuously during a session; finalised at session end
- **Contains:** Full audit trail — every file touched, every command run, every task result
- **How to retrieve:** Navigate to `/logs/` and open the file for the target date
- **Built in:** From Session 3 onward (auto-generated by orchestrator)
- **Status:** NOT YET CREATED (created by orchestrator at runtime)

---

## Memory Retrieval Guide

For the orchestrator and workers to use memory effectively:

| Query Type | Where to Look | How |
|---|---|---|
| "Did we do this before?" | `/memory/episodic-log.jsonl` | `grep "<keyword>"` or SQLite `WHERE outcome_summary LIKE '%keyword%'` |
| "Is there a skill for this?" | `/memory/skills-index.md` | Scan index for relevant slug, then read `/skills/<slug>.md` |
| "What does Smrithi prefer?" | `/memory/preferences.md` | Read the full file at session start |
| "What went wrong recently?" | `/system/FAILURE.md` | Scan for recent entries; check preventative actions |
| "What did we learn?" | `/memory/learnings.md` | Grep for keywords; read recent entries |
| "What happened this week?" | `/memory/weekly-summary.md` | Read latest section |

---

## Memory Integrity Rules

1. **Episodic log is append-only** — never edit or delete entries; only append
2. **Learnings are deduplicated** — before appending, check if the same insight already exists
3. **Preferences are overwrite-safe** — updating a preference replaces the old value (not append)
4. **SQLite is a cache** — if `agent.db` is corrupted or missing, rebuild from JSONL files; data is never lost
5. **All memory files are committed to git** (except `agent.db`) — the repo IS the persistent store
