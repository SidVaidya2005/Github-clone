# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install
npm start                    # Alias for: node index.js start (defined in package.json scripts)
node index.js start          # Equivalent long form

# Custom VCS CLI (run from any target directory):
node index.js init           # Initialize a .apnaGit repo in cwd
node index.js add <file>     # Stage a file
node index.js commit <msg>   # Commit staged files (UUID-based commit IDs)
node index.js push           # Push commits to S3
node index.js pull           # Pull commits from S3
node index.js revert <id>    # Revert to a specific commit
```

## Environment Variables

Create a `backend/.env` file manually — no `.env.example` exists in this directory:

```
MONGODB_URI=<MongoDB connection string>
PORT=3002
S3_BUCKET=<S3 bucket name>
JWT_SECRET_KEY=<secret for signing JWTs>
```

AWS credentials must be configured via AWS CLI profile or `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`.

## Architecture

### Dual-purpose entry point (`index.js`)
`yargs` dispatches two modes from a single entry:
- `node index.js start` → boots Express + Socket.io HTTP server
- All other subcommands → run the local custom VCS against `.apnaGit/` in `process.cwd()`

### HTTP API
`routes/main.router.js` composes three sub-routers:
- `user.router.js` → `userController.js` — JWT + bcrypt auth
- `repo.router.js` → `repoController.js` — repository CRUD
- `issue.router.js` → `issueController.js` — issue tracking

Mongoose models: `User`, `Repository` (embedded `issues[]` refs + `content[]`), `Issue`.

Socket.io clients join rooms by `userID` via a `joinRoom` event.

### Custom VCS (apnaGit)
Local state lives in `.apnaGit/` relative to `process.cwd()`:
- `.apnaGit/commits/<uuid>/` — one directory per commit, files copied in
- `.apnaGit/staging/` — staged files awaiting commit
- `.apnaGit/config.json` — stores `{ bucket: S3_BUCKET }`

`push`/`pull` sync the commits directory to/from AWS S3 using `aws-sdk` v2 (`config/aws-config.js`). The bucket name in that file is hardcoded as `"insert_bucket_name"` — override via `S3_BUCKET` env var (read in `init.js`).

### Known issues
- `JWT_SECRET_KEY` env var is required — both `signup` and `login` return HTTP 500 without it.
- `userController.js` uses a raw `MongoClient` singleton (not Mongoose) and hard-codes `db("githubclone")`. All other controllers use Mongoose models. When adding user-related endpoints, match the existing raw driver pattern in `userController.js`.
- `config/aws-config.js` hard-codes region `ap-south-1` and bucket `"insert_bucket_name"`. The `S3_BUCKET` env var only takes effect at `init` time (written to `config.json`); subsequent push/pull read from that file.
- `authMiddleware.js` and `authorizeMiddleware.js` are both empty — no routes are protected.
- `commit.json` in the project root is a leftover apnaGit artifact (not a config file).
