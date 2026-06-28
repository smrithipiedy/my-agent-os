# Architecture Decisions: Smrithi Agent OS

## Decision Log

### ADR-001
| Field | Value |
|---|---|
| **Date** | 2026-06-28 |
| **Session** | Session 1 |
| **Status** | ACCEPTED |
| **Context** | The agent OS needs a language for implementing workers and the orchestrator. Smrithi's stack includes both Python and JavaScript/TypeScript. The Gemini API has SDKs for both. |
| **Decision** | Python for all v1 workers and the orchestrator |
| **Reasoning** | Gemini Python SDK is first-class and well-documented. Python `pathlib` and `json` stdlib handle all file I/O needs without dependencies. Python's synchronous execution model is simpler for a single-user system. Smrithi confirmed this preference explicitly in Session 2. |
| **Alternatives Considered** | Node.js/TypeScript: rejected because it would require managing two runtimes (Python for AI calls, Node for the web layer); adds complexity with no v1 benefit. |
| **Consequences** | All workers are Python scripts. The hub/dashboard (Milestone 2+) will use Node.js/Next.js for the frontend but call Python workers via subprocess or API. Python environment must be verified at session start. |
| **Superseded By** | None |

---

### ADR-002
| Field | Value |
|---|---|
| **Date** | 2026-06-28 |
| **Session** | Session 1 (confirmed Session 2) |
| **Status** | ACCEPTED |
| **Context** | The system needs persistent memory across sessions. Options: files only, SQLite, Supabase, Firebase Firestore. |
| **Decision** | JSONL files as primary store; SQLite as index cache (gitignored); Supabase deferred to v2 |
| **Reasoning** | Files are the lowest-friction persistence layer: no database setup, human-readable, git-diffable, zero external dependencies. SQLite provides fast querying without requiring a server. Supabase would be more powerful but requires network, credentials, and adds a point of failure. Smrithi explicitly confirmed file-only + SQLite for v1. |
| **Alternatives Considered** | Supabase: deferred — requires network and credentials, overkill for single-user v1. Firebase Firestore: rejected for same reasons plus more complex auth. Pure SQLite without JSONL: rejected — SQLite files are not human-readable and produce noisy git diffs. |
| **Consequences** | All memory is in files in this repo. SQLite is regeneratable. If the repo is lost, memory is lost (mitigated by git remote in v2). |
| **Superseded By** | None |

---

### ADR-003
| Field | Value |
|---|---|
| **Date** | 2026-06-28 |
| **Session** | Session 2 |
| **Status** | ACCEPTED |
| **Context** | The orchestrator could run tasks automatically (fully autonomous) or present a task list to Smrithi for approval before executing (guided). This is the autonomy level decision. |
| **Decision** | Guided autonomy mode: show task list for human approval before executing any task |
| **Reasoning** | v1 is a new system with untested workers. Guided mode prevents irreversible actions (shell commands, file writes) from happening without human review. Maps to the original prompt's "AUTONOMY LEVELS" — start at guided, not autonomous. Smrithi confirmed this preference explicitly. |
| **Alternatives Considered** | Fully autonomous: rejected for v1 — too risky before workers are proven. Supervised (auto-proceed on low-risk): possible in v2 once workers have a track record via evals. |
| **Consequences** | Every task run requires Smrithi to review and type `approve` before execution begins. This adds 1–2 minutes per goal but prevents data loss and unintended side effects. The approval prompt is designed to be scannable (not verbose) to minimize friction. |
| **Superseded By** | None (will be revisited for v2) |

---

### ADR-004
| Field | Value |
|---|---|
| **Date** | 2026-06-28 |
| **Session** | Session 2 |
| **Status** | ACCEPTED |
| **Context** | The system needs a goal input format. Options: structured YAML/JSON, a specific template, or free-form plain text. |
| **Decision** | Free-form plain text in `/inbox/goal.md` |
| **Reasoning** | Lowest friction for Smrithi. Goals are often exploratory — forcing structure upfront kills the creative flow. The orchestrator handles structuring via Gemini decomposition. Smrithi confirmed this preference explicitly. |
| **Alternatives Considered** | Structured YAML: rejected — too rigid for exploratory goals; forces thinking about schema before thinking about the goal. JSON template: rejected for same reason. |
| **Consequences** | The Gemini decomposition prompt must be robust enough to handle ambiguous or short goals. A goal like "fix the CSV bug" must be expanded into a meaningful task graph. This requires a good decomposition prompt (see PATTERN-001 in system/knowledge.md). |
| **Superseded By** | None |
