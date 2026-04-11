# Known Issues & Tech Debt

Consolidated list of bugs, incomplete features, and architectural gaps. Update this file when issues are fixed or new ones are found.

---

## Security

### No route protection
`middleware/authMiddleware.js` and `middleware/authorizeMiddleware.js` are both empty files. Every API route is publicly accessible — no JWT verification happens server-side. The JWT stored in `localStorage["token"]` is never sent in request headers and never validated.

**Fix needed:** Implement `authMiddleware` to verify `Authorization: Bearer <token>`, then apply it to all routes except `POST /signup` and `POST /login`.

### CORS wildcard
`cors({ origin: "*" })` accepts requests from any origin. Acceptable for local dev, not for production.

---

## Missing Features

### `/create` route doesn't exist
`Navbar.jsx` links to `/create` (Create a Repository). No route or component exists for this path. Clicking the link renders nothing.

### Follower/star counts are static
`Profile.jsx` hard-codes `"10 Follower"` and `"3 Following"`. The `User.followedUsers` and `User.starRepos` arrays exist in the model but are never populated or read by the profile page.

### `User.repositories` is never updated
When `POST /repo/create` succeeds, it does not add the new `repositoryID` to `User.repositories[]`. The array always stays empty.

### Socket.io push not implemented
The server joins clients to rooms but never emits any events. Real-time features are wired up but non-functional.

---

## Configuration Issues

### AWS region and bucket hardcoded
`config/aws-config.js` hard-codes region `ap-south-1` and bucket `"insert_bucket_name"`. Must be edited in source.

### `S3_BUCKET` env var only effective at `init` time
The bucket name is baked into `.apnaGit/config.json` at `init`. Changing `S3_BUCKET` later has no effect on push/pull until `init` is re-run (which overwrites history).

### `JWT_SECRET_KEY` not in `.env.example`
Both `signup` and `login` return HTTP 500 if `JWT_SECRET_KEY` is missing. It must be in `.env` but is easy to miss.

---

## Code Quality

### Dual database drivers
`userController.js` uses raw `MongoClient`; everything else uses Mongoose. This creates two separate connection pools and two different query APIs that must be maintained in parallel.

### apnaGit staging never clears
`commit` does not clear `.apnaGit/staging/` after committing. Files accumulate in staging indefinitely.

### `commit.json` bleeds on revert
`revert <id>` copies all files from the commit directory including `commit.json` into the working directory. This file must be deleted manually after each revert.

### `console.log` left in production paths
`repoController.js:81` logs `req.params` on every `GET /repo/user/:userID` request. `index.js:98-102` logs socket join events including the userId.
