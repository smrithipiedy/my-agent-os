# System Knowledge Base
**System:** my-agent-os
**Type:** Living document — append new entries; never delete old ones
**Updated:** Session 2, 2026-06-28

This is the structured technical knowledge base for the agent OS. It captures patterns that work, anti-patterns to avoid, stack decisions, and external dependency failure modes — drawn from Smrithi's real project history and updated each session.

---

## Section 1: Technical Patterns That Work

### PATTERN-001: Gemini API — Decomposition Prompt Structure
**Context:** Asking Gemini to decompose a goal into structured sub-tasks
**Pattern:**
```
System prompt: "You are a task decomposition agent. Given a goal, return a JSON array of 2-7 tasks. Each task must have: description (string), worker_type (one of: code_writer, shell_runner, web_searcher, file_editor, browser_agent), expected_output (string describing what a correct result looks like), skill_tags (array of strings). Return only valid JSON. No markdown fences."
User prompt: <goal text>
```
**Why It Works:** Explicit JSON-only instruction prevents Gemini from wrapping output in markdown code blocks. Restricting `worker_type` to an enum prevents hallucinated worker types. Requiring `expected_output` forces the model to specify success criteria upfront.
**Source:** Derived from CarbonWise and BallotIQ Gemini integration experience.

---

### PATTERN-002: Gemini API — Retry with Exponential Backoff
**Context:** Any Gemini API call that might hit rate limits or transient failures
**Pattern (Python):**
```python
import time
import google.generativeai as genai

def call_with_backoff(prompt, model_name="gemini-1.5-flash", max_attempts=3):
    delays = [1, 2, 4]
    for attempt, delay in enumerate(delays, 1):
        try:
            model = genai.GenerativeModel(model_name)
            response = model.generate_content(prompt)
            if response.candidates and response.candidates[0].content:
                return response.candidates[0].content.parts[0].text
            raise ValueError("Empty response from Gemini")
        except Exception as e:
            if attempt == max_attempts:
                raise
            print(f"Attempt {attempt} failed: {e}. Retrying in {delay}s...")
            time.sleep(delay)
```
**Why It Works:** Handles transient API failures, quota resets (which happen on minute boundaries), and empty responses. Learned from BallotIQ quota exhaustion (FAIL-003).

---

### PATTERN-003: Next.js — Server/Client Firebase Separation
**Context:** Using Firebase in a Next.js 14+ application with App Router
**Pattern:**
- `lib/firebase-client.ts` — imports `firebase/app`, `firebase/auth` (browser-only, used in client components only)
- `lib/firebase-admin.ts` — imports `firebase-admin` SDK (Node.js only, used in server components and route handlers)
- Never import from `firebase-client.ts` in a file that can be rendered server-side
**Why It Works:** Avoids `window is not defined` SSR crashes. Learned from CrowdSense Firebase failure (FAIL-002).

---

### PATTERN-004: AI Response Validation with Zod (TypeScript)
**Context:** Validating any AI-generated structured output before persisting or using it
**Pattern:**
```typescript
const AIResponseSchema = z.object({
  tasks: z.array(z.object({
    description: z.string().min(1),
    worker_type: z.enum(["code_writer", "shell_runner", "web_searcher", "file_editor", "browser_agent"]),
    expected_output: z.string().min(1),
  })).min(2).max(7),
});

const result = AIResponseSchema.safeParse(JSON.parse(rawAIResponse));
if (!result.success) {
  // log the ZodError, trigger fallback, do NOT proceed with invalid data
}
```
**Why It Works:** `safeParse` returns errors instead of throwing — keeps the system running. Strict field validation catches Gemini's occasional null/missing fields. Learned from CarbonWise Zod failure (FAIL-001).

---

### PATTERN-005: JSONL for Append-Only Logs
**Context:** Writing episodic memory and session logs that must be queryable and never lose data
**Pattern:**
```python
import json
from pathlib import Path
from datetime import datetime, timezone

def append_to_log(filepath: str, entry: dict):
    entry["_written_at"] = datetime.now(timezone.utc).isoformat()
    with open(filepath, "a", encoding="utf-8") as f:
        f.write(json.dumps(entry) + "\n")
```
**Why It Works:** JSONL (one JSON object per line) is append-only by nature, human-readable, grep-able, and loadable line-by-line without parsing the entire file. Works with git diffs cleanly.

---

### PATTERN-006: SQLite as Memory Index Cache
**Context:** Querying episodic memory for relevant past tasks without reading the entire JSONL file
**Pattern:**
- Primary store: `/memory/episodic-log.jsonl` (append-only, canonical)
- Index cache: `/.system/agent.db` (SQLite, regeneratable, gitignored)
- On session start: sync new JSONL lines into SQLite
- For retrieval: query SQLite; for audit: read JSONL
**Why It Works:** Files stay canonical (easy to audit, repair, version control). SQLite provides fast keyword and field-based lookup. SQLite file is gitignored so it doesn't pollute commits.

---

## Section 2: Anti-Patterns to Avoid

### ANTI-001: Silent Empty Returns from AI Calls
**Problem:** Catching an API error and returning an empty object/string instead of raising
**Effect:** The system appears to work but produces no value; errors go undetected for hours
**Rule:** Every AI call must have explicit handling for: empty response, null candidates, HTTP 429, and network timeout. Empty is never acceptable as a silent return. (Source: FAIL-003)

### ANTI-002: Importing Browser-Only SDKs Server-Side
**Problem:** Importing Firebase client SDK, browser crypto, or DOM APIs in Next.js shared modules
**Effect:** SSR crashes with `window is not defined`; deployment blocked
**Rule:** Always check if an SDK is isomorphic before importing in a shared module. Separate browser and server SDK files. (Source: FAIL-002)

### ANTI-003: Using AI Output Without Schema Validation
**Problem:** Trusting that Gemini always returns the expected structure and passing it directly to the database
**Effect:** Malformed data corrupts records; schema violations cause 500 errors that look like 400s
**Rule:** All AI API responses must be validated with a schema (Zod in TS, Pydantic in Python, or manual dict-check in Python) before any downstream use. (Source: FAIL-001)

### ANTI-004: Marking Tasks Done Without Evidence Files
**Problem:** Saying "task completed" in chat or handoff without all five evidence files existing
**Effect:** Session continuity breaks; next session cannot verify what was actually done
**Rule:** See Contract 1 — Definition of Done. Five files must exist. No exceptions. (Source: AGENTS.md rule 4)

### ANTI-005: Holding State in Chat Instead of Files
**Problem:** Keeping task state, decisions, or memory in the conversation context only
**Effect:** State is lost when the session ends; next session has no continuity
**Rule:** All state must be written to files. Chat is disposable. Files are canonical. (Source: AGENTS.md rule 6)

### ANTI-006: Single-Model AI with No Fallback
**Problem:** Depending on one Gemini model version with no fallback when it fails
**Effect:** System-wide failure when the primary model is unavailable or over quota
**Rule:** Always define a fallback chain. For Gemini: try `gemini-1.5-pro` first, fall back to `gemini-1.5-flash`, then to a cached offline response. (Source: FAIL-003 + CarbonWise model waterfall)

---

## Section 3: Stack Decisions and Why

| Decision | Choice | Why | Alternatives Rejected |
|---|---|---|---|
| Worker language | Python | Gemini Python SDK is first-class; aligns with Smrithi's confirmed preference | Node.js (would need separate runtime for workers vs. web) |
| Memory primary store | JSONL files | Append-only, human-readable, git-diffable, zero dependencies | SQLite (rejected as primary — not human-readable; kept as index cache) |
| Memory index | SQLite via `/.system/agent.db` | Fast keyword queries without reading full JSONL; gitignored | Supabase (rejected for v1 — requires network, credentials, added complexity) |
| Goal input format | Free-form plain text in `/inbox/goal.md` | Lowest friction for Smrithi; no schema to remember | Structured YAML (rejected — too rigid for exploratory goals) |
| Autonomy mode | Guided (task list shown for approval before execution) | Matches Smrithi's stated preference; prevents irreversible actions | Fully autonomous (rejected — too risky for v1; shell commands need human eyes) |
| AI provider | Google Gemini (`gemini-1.5-flash` primary) | Already in Smrithi's stack; key available; free tier sufficient for v1 | OpenAI (rejected — not in stack; costs more; separate key management) |
| Web framework (future UI) | Next.js | Already in Smrithi's stack; SSR + API routes in one project | Vite/React SPA (rejected — no server-side capabilities) |
| Secrets management | `.env.local` (gitignored) | Simple, works on Windows, no extra tooling | GitHub Secrets (rejected for v1 — only useful for CI/CD, not local dev) |

---

## Section 4: External Dependencies and Their Failure Modes

| Dependency | Used For | Known Failure Modes | Mitigation |
|---|---|---|---|
| Gemini API (`google-generativeai`) | Goal decomposition, worker execution, verification | Quota exhaustion (HTTP 429), empty candidates, safety filter blocks, model version changes | Exponential backoff retry (3 attempts); fallback to `gemini-1.5-flash` if `pro` fails; explicit null-check on `candidates[0].content` |
| Python `pathlib` / `json` | File I/O, JSON parsing | Permission errors, disk full, malformed JSON | Wrap all file writes in try/except; validate JSON before writing; check disk space |
| SQLite (`sqlite3` stdlib) | Memory index cache | Database locked (concurrent access), file corruption | Single writer at a time (session-scoped); database is regeneratable from JSONL |
| Git | Version control, session commits | Merge conflicts (unlikely — single user), large binary files, commit sign failures | Single user eliminates merge conflicts; gitignore binary and secret files; no GPG signing required for v1 |
| Windows PowerShell | Shell command execution | Execution policy blocks scripts, path separator differences (`\` vs `/`) | Use `pathlib.Path` in Python (handles separators); document PowerShell execution policy requirements in setup |
| `.env.local` | API key access | File missing, wrong key name, key expired | Validate at session start: `python -c "import os; assert 'GEMINI_API_KEY' in os.environ"` after loading `.env.local` |
| Node.js / npm | Future UI builds | Version conflicts, `node_modules` drift | Pin Node version in `.nvmrc`; add `node_modules/` to `.gitignore` |
