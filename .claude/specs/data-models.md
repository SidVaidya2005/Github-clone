# Data Models

MongoDB database name: `"githubclone"` (hard-coded in `userController.js`).

---

## User

Defined in: `backend/models/userModel.js` (Mongoose schema)
Also accessed via raw `MongoClient` in `userController.js` — collection name: `"users"`

```
User {
  _id: ObjectId
  username: String (required, unique)
  email: String (required, unique)
  password: String (bcrypt hash)
  repositories: [ObjectId → Repository]
  followedUsers: [ObjectId → User]
  starRepos: [ObjectId → Repository]
  createdAt: Date (timestamps)
  updatedAt: Date (timestamps)
}
```

**Note:** `repositories` array on the User document is **not** automatically updated when a `Repository` is created via `POST /repo/create`. The two are only loosely linked; the `Repository.owner` field is the authoritative reference.

---

## Repository

Defined in: `backend/models/repoModel.js`

```
Repository {
  _id: ObjectId
  name: String (required, unique)
  description: String
  content: [String]          — append-only array (PUT /repo/update pushes to it)
  visibility: Boolean        — true = public, false = private (no default set)
  owner: ObjectId → User (required)
  issues: [ObjectId → Issue]
  createdAt: Date (timestamps)
  updatedAt: Date (timestamps)
}
```

**Note:** `content` stores raw strings (file content or notes) appended over time. There is no versioning within the document — each `PUT /repo/update` call pushes one new string entry.

---

## Issue

Defined in: `backend/models/issueModel.js`

> Read the model file directly for current field definitions — the issue schema is the least stable part of the codebase.

---

## Relationships

```
User ──< Repository   (Repository.owner → User._id)
Repository ──< Issue  (Repository.issues[] → Issue._id)
User ──< Repository   (User.starRepos[] → Repository._id)  [not maintained by API]
User ──< User         (User.followedUsers[] → User._id)     [not maintained by API]
```

Populate calls in `repoController.js` use `.populate("owner").populate("issues")` — these work via Mongoose refs. The `userController.js` raw MongoClient queries do **not** populate refs.
