# /agents — Agent Profiles and Behavior Packs

## Purpose
Agents are higher-order orchestrators that can manage multi-step workflows involving multiple workers. Where a worker does one thing (write code, run a shell command), an agent can string together a sequence of steps, make decisions based on intermediate results, and adapt its plan as it goes.

In v1, there is one orchestrator agent (`/orchestrator/run.py`). As the system grows, specialized agents for specific domains (research, coding, project management) will live here.

## What Goes Here
- Agent profile files (YAML or Markdown): define the agent's name, capabilities, allowed worker types, prompt templates, and autonomy level
- Behavior packs: sets of rules and prompt modifiers that change how an agent behaves in a given context (e.g., "cautious mode" vs. "fast mode")
- Agent-specific prompt templates (referenced by the orchestrator when building Gemini calls)

## Planned Agents (v1 and beyond)

| Agent | File | Purpose |
|---|---|---|
| Orchestrator | `/orchestrator/run.py` | Main session agent — reads goal, decomposes, routes, verifies |
| Code Agent | `agents/code-agent.yaml` (future) | Specialized for multi-file code generation tasks |
| Research Agent | `agents/research-agent.yaml` (future) | Searches web, reads papers, synthesizes findings |
| Review Agent | `agents/review-agent.yaml` (future) | Code review, security audit, quality check |

## What Does NOT Go Here
- Worker implementations — those live in `/workers/`
- Memory files — those live in `/memory/`
- Task files — those live in `/tasks/`

## Current State
v1: No agent profile files yet. The orchestrator's behavior is defined directly in `/orchestrator/run.py`. This directory is created now to reserve the namespace for future use.

## Rules
1. Every agent profile must declare: `name`, `version`, `capabilities`, `allowed_worker_types`, `autonomy_level`, `prompt_template`
2. Agents must not exceed their declared `allowed_worker_types` — this is the sandboxing mechanism
3. Agent behavior changes must be reflected in the profile file and committed before the change takes effect
4. Autonomy level must be one of: `guided` (shows plan for approval), `supervised` (auto-proceeds on low-risk tasks), `autonomous` (fully self-directed — v1 does NOT use this)
