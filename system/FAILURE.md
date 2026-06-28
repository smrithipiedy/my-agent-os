# Failure Log
**System:** my-agent-os
**Type:** Living document — append new entries; never delete old ones
**Updated:** Session 2, 2026-06-28

This file records every failure encountered across all sessions. Entries are written as they occur — not after the fact. Every failure must have a preventative action added to avoid the same failure in future sessions.

---

## Template

```markdown
### FAIL-<NNN>
**Date:** YYYY-MM-DD
**Session:** Session N
**Component:** <which part of the system failed>
**What Failed:** <one sentence>
**Why It Failed:** <root cause — be specific, not vague>
**Impact:** <what did not happen as a result>
**What Was Tried:** <list of recovery attempts>
**Resolution:** <what actually fixed it>
**Preventative Action Added:** <what was changed in rules/code/docs to prevent recurrence>
**Status:** RESOLVED | UNRESOLVED | MONITORING
```

---

## Failure Log

### FAIL-001
**Date:** 2026-06-21
**Session:** CarbonWise — Production Hardening (external project, pre-agent-os)
**Component:** API validation layer (Zod schema)
**What Failed:** AI-generated carbon footprint data passed through the API without schema validation, causing malformed data to reach the database and corrupt user records.
**Why It Failed:** The Zod schema was defined but not applied to AI response payloads — only to incoming user request bodies. The assumption was that Gemini API responses were always well-structured, which is false. The AI occasionally omitted fields or returned unexpected types under load.
**Impact:** Several user profiles had `null` values written to numeric fields. Two database records required manual correction. One API request that should have returned a 400 triggered an unhandled 500 instead.
**What Was Tried:**
- Added `console.log` to inspect AI response shape → confirmed missing fields
- Attempted to add `.optional()` to all fields → masked the real issue instead of catching it
- Restarted the API server → no effect
**Resolution:** Applied Zod `.safeParse()` to the Gemini API response immediately after receiving it, before any downstream processing. Added a model fallback waterfall: if primary Gemini model fails validation, retry with a secondary model. If secondary also fails, return a structured error to the user instead of writing to the database.
**Preventative Action Added:**
- Rule added to `/rules/`: "All AI API responses MUST be validated with a schema before being persisted or used in downstream logic. `safeParse` preferred over `parse` so errors are handleable."
- Eval test case added: `evals/test-ai-response-validation.json` — submits a goal that involves AI output, verifies the response was schema-validated before writing
**Status:** RESOLVED

---

### FAIL-002
**Date:** 2026-06-19
**Session:** CrowdSense — Real-time Navigation (external project, pre-agent-os)
**Component:** Firebase SDK — Server-side rendering in Next.js
**What Failed:** Firebase client SDK was imported in a Next.js server component, causing a `window is not defined` error and crashing the entire SSR render pipeline.
**Why It Failed:** Firebase client SDK directly references browser globals (`window`, `navigator`, `indexedDB`) that do not exist in the Node.js server environment. The assumption was that Firebase SDK was isomorphic — it is not. The `firebase/app` client package is browser-only.
**Impact:** Every page that imported the Firebase module failed to server-render. The application fell back to client-only rendering, breaking SEO and causing a blank initial page flash. Deployment blocked for 4 hours.
**What Was Tried:**
- Moved imports inside `useEffect` → fixed the crash but caused hydration mismatches
- Used `dynamic(() => import(...), { ssr: false })` → partially worked for components but not for server actions
- Tried `typeof window !== 'undefined'` guard → masked the issue, did not solve it architecturally
**Resolution:** Separated Firebase into two initialization files: `lib/firebase-client.ts` (browser-only, imported only in client components) and `lib/firebase-admin.ts` (Node.js SDK, imported only in server components and API routes). Added ESLint rule to prevent cross-importing.
**Preventative Action Added:**
- Rule added to `/rules/`: "Never import browser-only SDKs (Firebase client, browser crypto, DOM APIs) in server-side code. Always check SDK docs for 'isomorphic' status before importing in a Next.js shared module."
- `/system/knowledge.md` updated with: "Firebase in Next.js — use `firebase-admin` server-side, `firebase/app` client-side. Never mix."
**Status:** RESOLVED

---

### FAIL-003
**Date:** 2026-06-20
**Session:** BallotIQ — Adaptive Learning Platform (external project, pre-agent-os)
**Component:** Gemini API — quota management
**What Failed:** The BallotIQ Gemini API integration hit rate limits during a sustained load test, causing all AI-assisted learning path generation to silently return empty responses. Users received blank recommendation cards with no error message.
**Why It Failed:** The application had a single API key with no quota monitoring. No retry logic existed. No fallback response was defined for when the API was unavailable. The `quota exhausted` response from Gemini (HTTP 429) was caught by a bare `catch` block that returned an empty object, which the frontend rendered as empty cards instead of showing an error state.
**Impact:** Feature appeared to be working (no crash, no 500) but was producing zero value. The issue went undetected for ~2 hours of live user sessions. Trust in the AI feature was damaged.
**What Was Tried:**
- Increased the quota limit in Google Cloud Console → quota already at the free tier maximum
- Added `console.error` in catch block → revealed the silent failure pattern
- Waited for quota reset → system recovered, but the underlying pattern remained
**Resolution:**
1. Added explicit HTTP 429 handling that returns a structured fallback response (pre-computed offline recommendations)
2. Added exponential backoff retry (3 attempts, 1s/2s/4s delays)
3. Added quota monitoring via a simple counter written to a file — if counter > threshold, switch to offline mode proactively
4. Frontend now renders an explicit "AI recommendations unavailable, showing cached suggestions" state instead of empty cards
**Preventative Action Added:**
- Rule added to `/rules/`: "Every Gemini API call MUST have: (1) explicit 429 handling, (2) at least one retry with backoff, (3) a defined fallback response for when the API is unavailable. Silent empty returns are forbidden."
- Rule added to `/rules/`: "AI features MUST degrade gracefully — always show something useful to the user even when the AI is down."
- `/system/knowledge.md` updated with: "Gemini quota limits — free tier resets every minute. For production, use exponential backoff + offline fallback. Monitor quota with a counter file."
- The orchestrator in this system will implement the same 3-attempt backoff pattern for all Gemini calls.
**Status:** RESOLVED
