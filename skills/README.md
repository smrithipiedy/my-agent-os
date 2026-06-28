# /skills — Reusable Skill Files

## Purpose
Skills are reusable patterns, code templates, and prompt strategies extracted from completed, verified tasks. When a new task is similar to something the system has done before, it should find the relevant skill and reuse it — reducing token cost, improving consistency, and avoiding re-learning what already works.

One skill file = one repeatable task type.

## What Goes Here
- One Markdown file per skill (e.g., `scaffold-nextjs-app.md`, `write-python-csv-reader.md`)
- Each file contains: the skill pattern, example usage, source task ID, worker type, and tags
- `_index.md` — the skills index (also maintained in `/memory/skills-index.md`)

## Skill File Format

```markdown
# Skill: <human-readable name>
**Slug:** <kebab-case-identifier>
**Worker Type:** <code_writer|shell_runner|web_searcher|file_editor|browser_agent>
**Tags:** [tag1, tag2, tag3]
**Source Task:** <task_id>
**Last Used:** YYYY-MM-DD
**Use Count:** N

## When to Use This Skill
<Description of the type of task this skill applies to>

## Pattern
<The actual reusable pattern — code template, prompt, shell script, etc.>

## Example Usage
<A concrete example showing how to apply this pattern>

## Known Limitations
<What this skill does NOT handle; edge cases to watch for>

## Changelog
- YYYY-MM-DD: Created from task <task_id>
```

## What Does NOT Go Here
- Completed task outputs — those live in `/outputs/`
- Worker implementations — those live in `/workers/`
- General documentation — that lives in `/docs/`

## Current State
v1: No skills yet. Skills are extracted from completed tasks starting in Milestone 1. By Milestone 2, at least 3 skills should exist and the system should reference them before generating from scratch.

## Rules
1. Skills are extracted only from tasks with `verified: true` — unverified outputs do not become skills
2. Each skill must have a slug that is unique across all skill files
3. When a worker uses a skill, it must set `skill_used: <slug>` in the task output
4. Skills are updated (not replaced) when a better version of the same pattern is found
5. Skills index (`/memory/skills-index.md`) must be updated whenever a skill is added or modified
