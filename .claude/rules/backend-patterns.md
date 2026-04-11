# Backend Patterns

Patterns observed across `backend/`. Follow these when adding or modifying backend code.

## Adding a new route

1. Create `controllers/<noun>Controller.js` with named async exports
2. Create `routes/<noun>.router.js` — mount handlers on an Express Router
3. Import and `use()` the new router in `routes/main.router.js`

No route prefix is added in `main.router.js` — all routes are flat on `/`.

## Controller function signature

```js
async function doSomething(req, res) {
  const { param } = req.params;
  const { field } = req.body;

  try {
    // ...
    res.json({ message: "...", data });
  } catch (err) {
    console.error("Error during X : ", err.message);
    res.status(500).send("Server error");
  }
}
```

- `console.error` logs always include the operation context and `err.message`
- 4xx errors return `res.json({ message: "..." })` or `res.json({ error: "..." })`
- 5xx errors use `res.send("Server error")` (plain text, no JSON)

## User-related queries (raw MongoClient)

```js
async function connectClient() {
  if (!client) {
    client = new MongoClient(uri, { ... });
    await client.connect();
  }
}

// In each function:
await connectClient();
const db = client.db("githubclone");
const col = db.collection("users");
```

Use `new ObjectId(id)` from `mongodb` when querying by `_id`:
```js
var ObjectId = require("mongodb").ObjectId;
await col.findOne({ _id: new ObjectId(id) });
```

## Repo/Issue queries (Mongoose)

```js
const Thing = require("../models/thingModel");

// Find with population
const items = await Thing.find({}).populate("owner").populate("issues");

// Find by id (returns array — not findById)
const item = await Thing.find({ _id: id }).populate("owner");

// Save pattern for updates
const item = await Thing.findById(id);
item.field = newValue;
await item.save();
```

Note: `repoController.js` uses `Repository.find({ _id: id })` (returns array) rather than `findById` for single-item lookups. Match this pattern in repo/issue controllers.

## ObjectId validation

Always validate ObjectIds before querying to avoid Mongoose cast errors:
```js
if (!mongoose.Types.ObjectId.isValid(id)) {
  return res.status(400).json({ error: "Invalid ID!" });
}
```

## Environment variables

Access via `process.env.VAR` directly — `dotenv.config()` runs once at startup in `index.js`. Do not call `dotenv.config()` inside controllers.

## Socket.io events

Currently only `joinRoom` is handled (client → server). To push an event to a user:
```js
// io is the Server instance created in index.js — not currently exported
// To use io in controllers, pass it as a parameter or attach to app: app.set("io", io)
io.to(userID).emit("eventName", payload);
```
`io` is not currently accessible outside `index.js`. Pass it down or attach to `app` before adding server-push features.
