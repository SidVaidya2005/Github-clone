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
| `POST` | `/issue/create` | `{ ... }` | Created issue |
| `GET` | `/issue/all` | — | `Issue[]` |
| `GET` | `/issue/:id` | — | `Issue` |
| `PUT` | `/issue/update/:id` | `{ ... }` | Updated issue |
| `DELETE` | `/issue/delete/:id` | — | Deleted confirmation |

> Issue model fields: see `backend/models/issueModel.js`.

---

## Root

| Method | Path | Response |
|--------|------|----------|
| `GET` | `/` | `"Welcome!"` |
