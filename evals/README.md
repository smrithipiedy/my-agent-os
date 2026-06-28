# /evals — Eval Harness, Test Cases, Results History

## Purpose
Evals are the quality gate for the agent OS. They are structured test cases that verify the system produces the correct output for a known input. Evals run after changes to workers, orchestrator, or skills to confirm nothing regressed. They are the automated equivalent of the manual verification described in REQUIREMENTS.md.

Evals are not unit tests for code. They are end-to-end assertions about the system's behavior: "given this goal, did the system produce output that satisfies these criteria?"

## What Goes Here
- Test case files: `/evals/cases/<id>.json` — each is a goal + expected output + pass/fail criteria
- Eval runner: `/evals/run_eval.py` — submits each test case through the pipeline and checks results
- Results history: `/evals/results-YYYYMMDD.jsonl` — one JSONL line per test case per run
- Baseline file: `/evals/baseline.json` — the last known-good pass rates per test case

## Test Case File Format

```json
{
  "eval_id": "eval-001",
  "description": "Basic code generation — CSV reader",
  "goal": "Write a Python function that reads a CSV file and returns a list of column names",
  "expected_output_criteria": [
    "Output must contain valid Python code",
    "Code must use the csv module or pandas",
    "Function must accept a filepath parameter",
    "Function must return a list"
  ],
  "worker_type": "code_writer",
  "should_pass": true,
  "tags": ["code", "python", "basic"]
}
```

## What Does NOT Go Here
- Actual task files — those live in `/tasks/`
- Worker implementations — those live in `/workers/`
- Memory files — those live in `/memory/`

## Current State
v1: No eval cases yet. Eval harness is Milestone 3. The directory is created now to reserve the namespace. First 5 eval cases will be defined in Session 3 alongside the first Milestone 1 test run.

## Rules
1. Every eval case must have a `should_pass` field — at least one test case must be a known-bad input that should fail (regression detection)
2. Eval results must be written to `/evals/results-YYYYMMDD.jsonl` — never overwrite; always append a new daily file
3. If a test case that previously passed now fails, it must be flagged in the session log as a regression
4. The eval runner must exit with a non-zero code if any `should_pass: true` case fails — this stops session auto-proceed
5. Eval cases must be updated when the system's expected behavior changes — do not let eval cases drift from reality
