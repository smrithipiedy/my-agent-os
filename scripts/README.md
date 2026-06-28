# /scripts — Utility Scripts

## Purpose
Scripts are standalone utilities that support the agent OS infrastructure — they are not part of the main orchestration pipeline. They handle setup, maintenance, data migration, and one-off admin tasks.

## What Goes Here
- `setup.py` — environment setup: verify Python version, install dependencies, create required directories, validate `.env.local`
- `rebuild-db.py` — rebuild `/.system/agent.db` from JSONL files (use when the SQLite cache is corrupted)
- `export-memory.py` — export memory files to a readable format for review
- `health-check.py` — verify the system is in a valid state: required files exist, no orphaned tasks, memory is consistent
- `clean-outputs.py` — archive old outputs to free space (with human approval; never delete)
- Any one-off migration or data-fix script (name with date prefix: `YYYYMMDD-description.py`)

## What Does NOT Go Here
- Worker implementations — those live in `/workers/`
- Orchestrator logic — that lives in `/orchestrator/`
- Eval runner — that lives in `/evals/`

## Current State
v1: `setup.py` is the first script to build (Session 3). Other scripts are added as needed.

## Rules
1. Every script must have a docstring at the top explaining what it does, what it requires, and what it changes
2. Scripts must be safe to run multiple times (idempotent) unless explicitly marked `[DESTRUCTIVE: run once only]`
3. Destructive scripts must print a confirmation prompt before doing anything irreversible
4. No hardcoded secrets in scripts — always read from `.env.local` or environment variables
5. Every script run must be logged to the current session log
