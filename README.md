# Github Clone

A full-stack GitHub replica built with the MERN stack, featuring a **custom version control system** (apnaGit) implemented from scratch with AWS S3 as remote storage.

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite, React Router v6, Primer React, Axios |
| Backend | Node.js, Express, Socket.io, Mongoose, MongoDB |
| Auth | JWT (jsonwebtoken), bcryptjs |
| VCS Remote | AWS S3 (aws-sdk v2) |
| Testing | Vitest, Testing Library |

## Project Structure

```
.
├── backend/        Node.js/Express HTTP server + apnaGit CLI
└── frontend/       React 18 SPA (Vite)
```

The two apps are fully independent — they share no code and communicate exclusively over HTTP on `localhost:3002`.

## Features

- **User auth** — signup, login, profile management (JWT-based)
- **Repository management** — create, update, toggle visibility, delete repos
- **Issue tracking** — create, update, and close issues per repository
- **Contribution heatmap** — activity visualization on the profile page
- **Real-time foundation** — Socket.io rooms wired up, ready for server-push events
- **apnaGit VCS** — a custom Git-like CLI for local versioning and S3-backed push/pull

## Getting Started

### Prerequisites

- Node.js 18+
- MongoDB (local or Atlas)
- AWS account with an S3 bucket (for VCS push/pull)

### Backend

```bash
cd backend
npm install
```

Create `backend/.env`:

```env
MONGODB_URI=<your MongoDB connection string>
PORT=3002
S3_BUCKET=<your S3 bucket name>
JWT_SECRET_KEY=<secret for signing JWTs>
```

AWS credentials must be available via `~/.aws/credentials` or `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` environment variables.

```bash
npm start      # starts the Express server on port 3002
```

### Frontend

```bash
cd frontend
npm install
npm run dev    # starts Vite dev server on http://localhost:5173
```

The frontend hardcodes all API calls to `http://localhost:3002` — the backend must run on that port.

## apnaGit — Custom VCS

The backend doubles as a local VCS CLI. Run commands from the directory you want to version:

```bash
node index.js init              # initialise a .apnaGit/ repo in cwd
node index.js add <file>        # stage a file
node index.js commit <message>  # commit staged files (UUID-based commit IDs)
node index.js push              # push commits to S3
node index.js pull              # pull commits from S3
node index.js revert <id>       # restore working directory to a commit
```

Local state is stored in `.apnaGit/` relative to the project directory:

```
.apnaGit/
  config.json        { "bucket": "<S3_BUCKET at init time>" }
  staging/           staged files awaiting commit
  commits/
    <uuid>/          one directory per commit
      <files>
      commit.json    { "message": "...", "date": "..." }
```

> Note: The S3 bucket is baked into `config.json` at `init` time. Changing `S3_BUCKET` later has no effect — edit `config.json` manually or re-run `init`.

## API Overview

Base URL: `http://localhost:3002`

| Resource | Endpoints |
|---|---|
| Users | `POST /signup`, `POST /login`, `GET /userProfile/:id`, `PUT /updateProfile/:id`, `DELETE /deleteProfile/:id` |
| Repositories | `POST /repo/create`, `GET /repo/all`, `GET /repo/:id`, `GET /repo/user/:userID`, `PUT /repo/update/:id`, `PATCH /repo/toggle/:id`, `DELETE /repo/delete/:id` |
| Issues | `POST /issue/create`, `GET /issue/all`, `GET /issue/:id`, `PUT /issue/update/:id`, `DELETE /issue/delete/:id` |

Full request/response shapes are documented in [`.claude/specs/api-reference.md`](.claude/specs/api-reference.md).

## Frontend Scripts

```bash
npm run dev      # development server
npm run build    # production build
npm run preview  # serve production build locally
npm run lint     # ESLint
npm run test     # Vitest
```

## Known Limitations

- Auth middleware is not implemented — all routes are publicly accessible
- `User.repositories` is not updated on repo creation
- apnaGit staging directory is not cleared after commits
- `revert` copies `commit.json` into the working directory (delete it manually)
- Follower/following counts are hardcoded on the profile page
- `/create` route (new repository) is linked in the navbar but not yet implemented
