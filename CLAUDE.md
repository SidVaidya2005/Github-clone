# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A MERN-stack GitHub clone with a **custom version control system** ("apnaGit") implemented from scratch. The project has two independent apps:

- `backend-main/` — Node.js/Express server + CLI tool
- `frontend-main/` — React 18 + Vite SPA

## Commands

### Backend
```bash
cd backend-main
npm install
node index.js start          # Start the HTTP server (default port 3000)

# Custom VCS CLI (run from any project directory):
node index.js init           # Initialize a .apnaGit repo in cwd
node index.js add <file>     # Stage a file
node index.js commit <msg>   # Commit staged files (UUID-based commit IDs)
node index.js push           # Push commits to S3
node index.js pull           # Pull commits from S3
node index.js revert <id>    # Revert to a commit
```

### Frontend
```bash
cd frontend-main
npm install
npm run dev      # Start Vite dev server
npm run build    # Production build
npm run lint     # ESLint
npm run test     # Vitest
```

## Architecture

### Backend dual-purpose entry point
`backend-main/index.js` serves two roles via `yargs`:
- `node index.js start` — boots the Express/Socket.io HTTP server
- All other subcommands (`init`, `add`, `commit`, `push`, `pull`, `revert`) — run the custom local VCS, operating on a `.apnaGit/` directory in the current working directory

The custom VCS stores state locally: `.apnaGit/commits/<uuid>/` for each commit, `.apnaGit/staging/` for staged files, `.apnaGit/config.json` for config. Push/pull sync commit directories to/from AWS S3.

### Backend HTTP API
Routes are split across three routers composed in `routes/main.router.js`:
- `user.router.js` → `userController.js` — auth (JWT + bcrypt)
- `repo.router.js` → `repoController.js` — CRUD on repositories
- `issue.router.js` → `issueController.js` — issue tracking

Mongoose models: `User`, `Repository` (with embedded `issues[]` refs and `content[]`), `Issue`.

Socket.io is initialized in the server for real-time features; clients join rooms by `userID`.

### Frontend auth flow
`authContext.jsx` provides `currentUser` (a userId string) via React Context, persisted in `localStorage` as `"userId"`. `Routes.jsx` guards all routes — unauthenticated users are redirected to `/auth`. The four routes are: `/` (Dashboard), `/auth` (Login), `/signup` (Signup), `/profile` (Profile).

### Known configuration issues
- **Port mismatch**: backend defaults to port `3000`, but frontend API calls are hardcoded to `http://localhost:3002`. Set `PORT=3002` in the backend `.env`.
- **AWS S3**: `config/aws-config.js` has the bucket name hardcoded as `"insert_bucket_name"` and region hardcoded to `ap-south-1`. Update this file or use the `S3_BUCKET` env var pattern from `initRepo`.

## Environment Variables (backend)

```
MONGODB_URI=<your MongoDB connection string>
PORT=3002
```

AWS credentials must be configured separately (via AWS CLI profile or environment variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`).
