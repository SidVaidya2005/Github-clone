# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A MERN-stack GitHub clone with a **custom version control system** ("apnaGit") implemented from scratch. The project has two independent apps:

- `backend/` — Node.js/Express HTTP server + apnaGit CLI tool → see [`backend/CLAUDE.md`](backend/CLAUDE.md)
- `frontend/` — React 18 + Vite SPA → see [`frontend/CLAUDE.md`](frontend/CLAUDE.md)

Each subdirectory has its own `CLAUDE.md` with commands, env vars, and app-specific architecture. Read those files when working inside a single app. This file covers only cross-cutting concerns.

## Reference docs (`.claude/`)

| File | Purpose |
|------|---------|
| [`specs/architecture.md`](.claude/specs/architecture.md) | Full system architecture, component tree, router composition |
| [`specs/api-reference.md`](.claude/specs/api-reference.md) | All REST endpoints with request/response shapes |
| [`specs/data-models.md`](.claude/specs/data-models.md) | MongoDB schemas and their relationships |
| [`specs/apnagit-vcs.md`](.claude/specs/apnagit-vcs.md) | Custom VCS internals, storage layout, command mechanics |
| [`rules/code-style.md`](.claude/rules/code-style.md) | JS/JSX conventions, naming, async patterns |
| [`rules/design-system.md`](.claude/rules/design-system.md) | CSS tokens, typography, color palette, Primer overrides |
| [`rules/frontend-patterns.md`](.claude/rules/frontend-patterns.md) | React component patterns, auth fetch, routing |
| [`rules/backend-patterns.md`](.claude/rules/backend-patterns.md) | Express patterns, controller shape, DB access |
| [`rules/known-issues.md`](.claude/rules/known-issues.md) | Bugs, missing features, security gaps, tech debt |

## Quick Start

```bash
# Backend (terminal 1) — create backend/.env first (no .env.example exists, see backend/CLAUDE.md)
cd backend && npm install && npm start

# Frontend (terminal 2)
cd frontend && npm install && npm run dev
```

Backend must run on `PORT=3002` (frontend API calls are hardcoded to `http://localhost:3002`).

## Cross-cutting Architecture

### Data flow
Browser → HTTP (`http://localhost:3002`) → Express routes → Mongoose (MongoDB)
> Note: `Login.jsx`/`Profile.jsx` use axios; `Dashboard.jsx` uses native `fetch()`.

Real-time: frontend connects via Socket.io and emits `joinRoom <userId>` to subscribe to user-specific events.

### Auth
JWT issued by backend on login, stored as `userId` in `localStorage` by the frontend. `middleware/authMiddleware.js` exists but is **empty** — all API routes are currently unprotected. Do not assume auth is enforced server-side.

### apnaGit VCS
The backend doubles as a local CLI. `node index.js <cmd>` operates on `.apnaGit/` in whatever directory it is run from — completely independent of the HTTP server. Push/pull sync to AWS S3.
