# /projects — One Subfolder Per Project

## Purpose
Projects is where the agent OS does real work. Each project is a self-contained directory with its own goal, plan, task list, knowledge base, decisions log, status, and handoff. The agent OS reads from and writes to project directories when executing goals scoped to a specific project.

## What Goes Here
- One subfolder per active project (e.g., `/projects/smrithi-agent-os/`, `/projects/carbonwise/`)
- `_template/` — the template that every new project copies from
- Each project subfolder contains exactly the files defined in `_template/`

## Project Naming Convention
- Use kebab-case: `my-project-name`
- Use descriptive names, not codes: `smrithi-agent-os` not `proj-001`
- Add a date prefix for time-bounded projects: `2026-06-contest-submission`

## Active Projects

| Project | Folder | Status | Description |
|---|---|---|---|
| Agent OS | `smrithi-agent-os/` | ACTIVE | Building the system itself — the meta-project |

## What Does NOT Go Here
- System-level files — those live in `/system/`
- Memory — that lives in `/memory/`
- Shared skills — those live in `/skills/`
- Worker outputs — those live in `/outputs/`

## Rules
1. Every new project MUST be created by copying `_template/` — do not create project folders from scratch
2. Every project folder MUST contain all 8 files from the template (`project.md`, `plan.md`, `tasks.md`, `knowledge.md`, `decisions.md`, `status.md`, `handoff.md`, `artifacts/README.md`)
3. Project-specific tasks are tracked in `/projects/<project>/tasks.md` — the system-level `/tasks/active/` directory holds the JSON task files for the orchestrator
4. Completed projects are moved to `projects/archive/` — never deleted
