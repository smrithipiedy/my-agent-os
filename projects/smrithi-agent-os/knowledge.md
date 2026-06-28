# Project Knowledge: Smrithi Agent OS

## Technical Patterns That Work

### PATTERN-P001: Session Continuity via File Trio
**Context:** Every AI session in Antigravity IDE starts with no memory of previous sessions
**Pattern:** At the start of every session, always read three files in this exact order: `AGENTS.md` → `handoff.md` → `status.md`. This trio gives complete context without needing to read chat history.
**Why It Works:** `AGENTS.md` sets the rules, `handoff.md` tells you what was done and what to do next, `status.md` tells you the system's health. All three together provide enough context to resume any session cold. Proven effective in Sessions 1 and 2.

### PATTERN-P002: Parallel File Creation for Documentation
**Context:** Creating multiple independent files in the same session
**Pattern:** When creating files that do not depend on each other's content, invoke multiple `write_to_file` calls in the same tool batch. This reduces session time significantly.
**Why It Works:** Antigravity executes parallel tool calls concurrently. For documentation-heavy sessions (like this one), parallelism cuts session time by 60–70%.
**Note:** Do NOT parallelize files that reference each other — write the source file first, then the dependent file.

### PATTERN-P003: JSONL as the Canonical Memory Format
**Context:** Storing task history and episodic memory across sessions
**Pattern:** Use `.jsonl` (one JSON object per line) for all append-only logs. Use `json.dumps(entry) + "\n"` for writing, and read with `for line in f: entry = json.loads(line)`.
**Why It Works:** JSONL is append-safe (no need to read/parse the full file to add an entry), human-readable, grep-searchable, and produces clean git diffs (one line added per entry). Chosen over SQLite as primary store because it requires no database setup.

---

## Gotchas and Pitfalls

### GOTCHA-P001: PowerShell `&&` Not Supported as Statement Separator
**Symptom:** `run_command` with `&&` between commands fails with `ParserError: '&&' is not a valid statement separator`
**Cause:** PowerShell (unlike bash/zsh) does not support `&&` for conditional command chaining
**Fix:** Use `;` as the separator in PowerShell: `git add .; git commit -m "message"`. For conditional execution, use `if ($LASTEXITCODE -eq 0) { ... }` blocks.
**First encountered:** Session 1 — git commit attempt

### GOTCHA-P002: Antigravity Sessions Have No Cross-Session Memory
**Symptom:** Agent in new session has no knowledge of previous session's context, decisions, or files created
**Cause:** Each Antigravity session is a fresh context window; chat history is not preserved
**Fix:** The AGENTS.md → handoff.md → status.md read protocol IS the solution. All decisions, all file paths, all blockers must be written to files — never left in chat. Files are canonical; chat is disposable.
**Impact:** If handoff.md is not updated at session end, the next session cannot resume without re-reading all files.

---

## API and Service Notes

### Gemini API
**Version Used:** `gemini-1.5-flash` (primary), `gemini-1.5-pro` (fallback for complex tasks)
**Key Notes:**
- API key is in `.env.local` as `GEMINI_API_KEY` ✅
- Python SDK: `google-generativeai` (install: `pip install google-generativeai`)
- Always check `response.candidates[0].content` for null before accessing text — safety filters can return empty candidates
- Use `gemini-1.5-flash` for decomposition and verification (fast, cheap); use `gemini-1.5-pro` for code generation (better quality)
**Known Issues:**
- HTTP 429 on quota exhaustion — use exponential backoff (see PATTERN-001 in system/knowledge.md)
- Safety filter blocks with no content — handle with null check + fallback

### Git (via PowerShell)
**Key Notes:**
- Use `;` not `&&` for command chaining in PowerShell
- Line endings: Windows uses CRLF; git will warn about LF→CRLF conversion — this is normal and safe
- `.gitignore` already configured: `.env.local`, `.env`, `*.env`, `/.system/agent.db`, `__pycache__/`, `*.pyc`

---

## Environment and Configuration Notes

| Variable | File | Status | Notes |
|---|---|---|---|
| `GEMINI_API_KEY` | `.env.local` | ✅ Confirmed | Smrithi confirmed in handoff.md answers |
| Python version | System | ❓ Not verified | Verify in Session 3: `python --version` |
| `google-generativeai` | pip | ❓ Not installed | Install in Session 3: `pip install google-generativeai` |
| Node.js version | System | ❓ Not verified | Needed for future hub/dashboard |

**Loading `.env.local` in Python:**
```python
from dotenv import load_dotenv
import os
load_dotenv(".env.local")
api_key = os.getenv("GEMINI_API_KEY")
assert api_key, "GEMINI_API_KEY not found in .env.local"
```
Note: requires `pip install python-dotenv`
