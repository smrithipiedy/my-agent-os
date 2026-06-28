# Task List
<!-- 
INSTRUCTIONS: Every task must use this full schema. No shortcuts.
Tasks are the atomic unit of work — each one must be independently verifiable.
Add new tasks at the bottom. Never delete completed tasks — mark them done.

STATUS VALUES: pending | approved | in_progress | completed | verified | failed | waiting_approval | blocked
RISK LEVELS: low | medium | high | critical
PRIORITY: p1 (now) | p2 (this milestone) | p3 (future)
-->

## Active Tasks

### TASK-001
| Field | Value |
|---|---|
| **id** | TASK-001 |
| **goal_id** | goal-<!-- fill in --> |
| **project_id** | proj-<!-- fill in --> |
| **description** | <!-- What exactly should be done? --> |
| **skill_tags** | <!-- e.g., python, nextjs, gemini, firebase --> |
| **status** | pending |
| **depends_on** | none |
| **owner** | Smrithi P |
| **reviewer** | agent-verifier |
| **priority** | p1 |
| **risk_level** | low |
| **budget_limit** | <!-- max tokens or time, e.g., "2000 tokens" or "30 min" --> |
| **tokens_used** | 0 |
| **attempts** | 0 |
| **verification_plan** | <!-- How will we know this is done? What will we check? --> |
| **evidence** | <!-- Filled in after completion: link to result.md and verification.md --> |
| **artifacts** | <!-- Filled in after completion: list of files produced --> |
| **escalation_reason** | none |
| **created_at** | <!-- ISO 8601 timestamp --> |
| **updated_at** | <!-- ISO 8601 timestamp --> |

---

## Completed Tasks

<!-- Move tasks here when verified: true is written to their verification.md -->

---

## Failed Tasks

<!-- Move tasks here when verified: false and no more retries remain. Document why. -->

---

## Blocked Tasks

<!-- Move tasks here when a dependency is unresolved. Document what is blocking. -->
