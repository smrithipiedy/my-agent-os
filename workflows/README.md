# /workflows — Specialized Workflow Harnesses

## Purpose
Workflows are multi-step, domain-specific pipelines that go beyond a single goal → decompose → execute cycle. They are pre-defined sequences of tasks optimized for a specific repeated use case — like "scaffold a new Next.js project" or "do a weekly review of memory and skills."

Each workflow is a Python script that chains multiple orchestrator calls, with checkpoints and human approval gates built in for complex or risky steps.

## What Goes Here
- One Python file per workflow (e.g., `scaffold-nextjs.py`, `weekly-review.py`, `code-review.py`)
- A manifest file per workflow: `<workflow-name>.manifest.json` defining the workflow's steps, inputs, and expected outputs
- `_base_workflow.py` — base class for all workflows

## Planned Workflows

| Workflow | File | Purpose |
|---|---|---|
| New Project Scaffold | `scaffold-project.py` | Create a new project folder with all template files pre-filled |
| Weekly Review | `weekly-review.py` | Summarize the last 7 sessions, extract top learnings, suggest improvements |
| Code Review | `code-review.py` | Review a codebase against known patterns and anti-patterns |
| Dependency Audit | `dep-audit.py` | Check all dependencies for known issues and outdated versions |

## What Does NOT Go Here
- Individual worker implementations — those live in `/workers/`
- Generic orchestration — that lives in `/orchestrator/`
- Task templates — those live in `/projects/_template/`

## Current State
v1: No workflows yet. Reserved for Milestone 2+. The directory is created now to reserve the namespace.

## Rules
1. Every workflow must have a `manifest.json` that defines: `name`, `version`, `description`, `steps`, `inputs`, `outputs`, `approval_required` (bool)
2. Workflows that modify files outside `/projects/` or `/outputs/` must have `approval_required: true`
3. Each step in a workflow must correspond to a valid worker type in `/workers/registry.json`
4. Workflows must be restartable from any checkpoint — if a step fails, the next run skips completed steps
