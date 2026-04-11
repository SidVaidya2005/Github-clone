# API Reference

Base URL: `http://localhost:3002`

All request/response bodies are JSON. No authentication headers are currently enforced on any route.

---

## User

| Method | Path | Body / Params | Response |
|--------|------|---------------|----------|
| `POST` | `/signup` | `{ username, password, email }` | `{ token, userId }` |
| `POST` | `/login` | `{ email, password }` | `{ token, userId }` |
| `GET` | `/allUsers` | — | `User[]` |
| `GET` | `/userProfile/:id` | — | `User` |
| `PUT` | `/updateProfile/:id` | `{ email, password? }` | Updated `User` |
| `DELETE` | `/deleteProfile/:id` | — | `{ message }` |

**Notes:**
- `signup` and `login` both require `JWT_SECRET_KEY` env var — return HTTP 500 without it.
- `login` looks up by `email` (not `username`).
- `token` in responses is a JWT (1h expiry). Frontend stores it in `localStorage["token"]` but never sends it in headers — it is unused for authorization.

---

## Repository

| Method | Path | Body / Params | Response |
|--------|------|---------------|----------|
| `POST` | `/repo/create` | `{ owner, name, description?, visibility?, content?, issues? }` | `{ message, repositoryID }` |
| `GET` | `/repo/all` | — | `Repository[]` (populated owner + issues) |
| `GET` | `/repo/:id` | — | `Repository[]` (populated) |
| `GET` | `/repo/name/:name` | — | `Repository[]` (populated) |
| `GET` | `/repo/user/:userID` | — | `{ message, repositories[] }` |
| `PUT` | `/repo/update/:id` | `{ content, description }` | `{ message, repository }` |
| `PATCH` | `/repo/toggle/:id` | — | `{ message, repository }` — flips `visibility` boolean |
| `DELETE` | `/repo/delete/:id` | — | `{ message }` |

**Notes:**
- `PUT /repo/update/:id` **appends** `content` to the array — it does not replace it.
- `GET /repo/:id` uses `Repository.find({ _id: id })` and returns an **array**, not a single object.
- `owner` must be a valid MongoDB ObjectId string.

---

## Issue

| Method | Path | Body / Params | Response |
|--------|------|---------------|----------|
| `POST` | `/issue/create` | `{ title, description }` | Created `Issue` (HTTP 201) |
| `GET` | `/issue/all` | — | `Issue[]` |
| `GET` | `/issue/:id` | — | `Issue` |
| `PUT` | `/issue/update/:id` | `{ title, description, status }` | Updated `Issue` |
| `DELETE` | `/issue/delete/:id` | — | `{ message: "Issue deleted" }` |

**Issue schema:**
```
Issue {
  _id: ObjectId
  title: String (required)
  description: String (required)
  status: "open" | "closed"  (default: "open")
  repository: ObjectId → Repository (required)
  createdAt: Date
  updatedAt: Date
}
```

**Bugs in `issueController.js` (as of last audit):**
- `POST /issue/create` reads `repository` from `req.params.id` — but the route has no `:id` param, so `repository` is always `undefined`. Creation will fail Mongoose validation. Likely needs `req.body.repository` or a route like `/issue/create/:repoId`.
- `GET /issue/all` also reads `req.params.id` (undefined) — returns all issues unfiltered regardless of repository.
- `getAllIssues` and `deleteIssueById` are missing `await` on their Mongoose calls — both silently return without waiting for the DB operation.

---

## Root

| Method | Path | Response |
|--------|------|----------|
| `GET` | `/` | `"Welcome!"` |
