# /hub — Control Plane

## Purpose
The hub is the control plane for the agent OS. It will eventually expose a REST API and WebSocket layer that allows the orchestrator, workers, and any future UI dashboard to communicate in a structured way. In v1, the hub is minimal — it exists as a defined location for this layer to grow into.

## What Goes Here
- REST API server (future: `server.py` or `server.js`)
- WebSocket event broadcaster (future: real-time task status to dashboard)
- API route definitions and schemas
- Authentication middleware (single-user for now; key-based in future)
- Health check endpoint

## What Does NOT Go Here
- Worker logic — that lives in `/workers/`
- Memory read/write — that lives in `/memory/`
- Task definitions — those live in `/tasks/`
- Any file that is not part of the server/API layer

## Current State
v1: Not yet implemented. The orchestrator runs directly as a Python script (`/orchestrator/run.py`). The hub API will be added in Milestone 2 when the system needs a dashboard or external triggers.

## Rules
1. All API endpoints must be documented here before implementation
2. No secrets or API keys in hub code — load from `.env.local` only
3. All requests must be logged to `/logs/`
4. Hub must be startable with a single command: `python hub/server.py` or `npm run dev` (depending on implementation choice)
