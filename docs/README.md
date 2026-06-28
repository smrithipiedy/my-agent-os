# /docs — System Documentation

## Purpose
Docs is the reference library for the agent OS. It contains setup guides, architecture overviews, API references, and explanations of how the system works — written for Smrithi (and any future AI agent) to read at the start of a new domain of work.

## What Goes Here
- `setup.md` — how to set up the system from scratch on a new machine
- `architecture.md` — system architecture overview with diagrams
- `api-reference.md` — orchestrator and hub API reference (future)
- `troubleshooting.md` — common problems and how to fix them
- Domain-specific guides (e.g., `guides/gemini-integration.md`, `guides/next-js-patterns.md`)

## What Does NOT Go Here
- System contracts — those live in `/system/contracts.md`
- Project-specific docs — those live in `/projects/<project>/`
- Failure log — that lives in `/system/FAILURE.md`
- Knowledge base — that lives in `/system/knowledge.md`

## Current State
v1: No doc files yet. The directory is created to reserve the namespace. `setup.md` is the first file to be written (Session 3).

## Rules
1. Every guide must have: purpose, prerequisites, step-by-step instructions, and a troubleshooting section
2. Docs are written for a reader who has not seen the codebase before (including a future AI agent loading this repo cold)
3. Diagrams are preferred over long prose for architecture explanations
4. Docs must be updated when the system changes — stale docs are worse than no docs
