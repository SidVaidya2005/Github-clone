# System Architecture

## Overview

Two independent Node.js apps that share no code. They communicate exclusively over HTTP on `localhost:3002`.

```
browser
  │
  ├─ HTTP (axios / fetch) → backend:3002
  │     ├─ Express routes → Mongoose (MongoDB: "githubclone")
  │     └─ Socket.io → rooms keyed by userId
  │
  └─ Vite dev server :5173 (frontend only)
```

## Backend (`backend/`)

### Entry point dispatch (`index.js`)
`yargs` reads `process.argv` at startup and routes to one of two completely separate modes:

| Invocation | Mode |
|------------|------|
| `node index.js start` | HTTP server (Express + Socket.io) |
| `node index.js <vcs-cmd>` | apnaGit CLI — operates on `process.cwd()/.apnaGit/` |

The two modes share no runtime state. Running the server does NOT enable VCS commands and vice versa.

### HTTP server startup sequence
1. `dotenv.config()` — load `.env`
2. `mongoose.connect(MONGODB_URI)` — connect to MongoDB
3. Express middlewares: `bodyParser.json`, `express.json`, `cors({ origin: "*" })`
4. Mount `mainRouter` at `/`
5. Create `http.Server` → attach `socket.io`
6. Listen on `PORT` (default 3000, set to 3002 in `.env`)

### Router composition
```
mainRouter (routes/main.router.js)
  ├── userRouter   (routes/user.router.js)
  ├── repoRouter   (routes/repo.router.js)
  └── issueRouter  (routes/issue.router.js)
```
All routers mount directly on `/` — no prefix namespacing.

### Database access split (critical)
`userController.js` uses a raw `MongoClient` singleton and hard-codes `client.db("githubclone")`. All other controllers use Mongoose models. This split is intentional — do not migrate one without the other.

### Socket.io
Clients emit `joinRoom <userId>` after login. The server stores the last joined `userId` in a module-level `user` variable (not per-socket). Real-time push from server to client is possible but no events are currently emitted by the server.

## Frontend (`frontend/`)

### App shell
```
main.jsx
  └─ <AuthProvider>          (authContext.jsx)
       └─ <BrowserRouter>
            └─ <App>
                 └─ <ProjectRoutes>   (Routes.jsx)
```

### Route guard logic (`Routes.jsx`)
A single `useEffect` on `[currentUser, navigate, setCurrentUser]` handles three cases:
1. `localStorage.userId` exists but context is empty → hydrate context
2. No `localStorage.userId` and not on `/auth` or `/signup` → redirect to `/auth`
3. Has `localStorage.userId` and on `/auth` → redirect to `/`

### Component structure
```
src/
  components/
    Navbar.jsx           shared top nav
    navbar.css
    auth/
      Login.jsx          axios POST /login
      Signup.jsx
      auth.css
    dashboard/
      Dashboard.jsx      fetch() GET /repo/user/:id + /repo/all
      dashboard.css
    user/
      Profile.jsx        axios GET /userProfile/:id
      HeatMap.jsx        @uiw/react-heat-map
      profile.css
  authContext.jsx         React Context + localStorage hydration
  Routes.jsx              route definitions + auth guard
  App.jsx
  main.jsx
  index.css              CSS custom properties (design tokens)
```
