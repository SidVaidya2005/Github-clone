# Code Style

## General

- **No TypeScript** — all files are `.js` (backend) or `.jsx` (frontend). Do not introduce `.ts`/`.tsx`.
- **No test coverage requirement** — tests exist via Vitest but are not comprehensive. Don't block on tests for bug fixes.
- **CommonJS in backend** — `require()` / `module.exports`. No ESM in backend.
- **ESM in frontend** — `import`/`export`. Vite handles bundling.

## Backend (Node.js / Express)

### File naming
`camelCase.js` for all files. Controllers are `<noun>Controller.js`, routers are `<noun>.router.js`, models are `<noun>Model.js`.

### Async pattern
All controller functions are `async/await` with `try/catch`. Error responses follow this shape:
```js
res.status(4xx).json({ message: "..." })   // client errors
res.status(500).send("Server error")        // server errors (no JSON body)
```

### Controller exports
Named exports only — `module.exports = { fn1, fn2 }`.

### Environment access
Always use `process.env.VAR_NAME` directly. `dotenv.config()` is called once at the top of `index.js`.

### Database access rule
- User-related operations → use the existing `MongoClient` singleton in `userController.js`
- Repo/Issue operations → use Mongoose models

Do not mix these in a single controller.

## Frontend (React / JSX)

### Component conventions
- One component per file, default export, filename matches component name
- Functional components only — no class components
- Props are not typed (no PropTypes, no TypeScript)

### State and effects
```jsx
const [value, setValue] = useState(initialValue);
useEffect(() => { /* async fetch inside */ }, [dep]);
```
Async functions are defined inside `useEffect` and called immediately — not passed directly to `useEffect`.

### Auth access pattern
Always read the userId from context OR localStorage — never rely on only one:
```jsx
const { currentUser, setCurrentUser } = useAuth();
const userId = localStorage.getItem("userId"); // fallback in effects
```

### Navigation
Use `useNavigate()` for programmatic navigation. Use `window.location.href` only for full-page reloads after auth state changes (login/logout) — see `Login.jsx` and `Profile.jsx`.

### HTTP calls
- `axios` is used in auth (`Login.jsx`, `Signup.jsx`) and data pages (`Profile.jsx`)
- Native `fetch()` is used in `Dashboard.jsx`
- Do not mix within the same component; prefer `axios` for new code

### Styling
- Per-component CSS files colocated next to the component (`auth.css`, `profile.css`, etc.)
- Global resets and tokens in `src/index.css`
- CSS custom properties (never hardcode colors, radii, shadows — see `design-system.md`)
- No CSS modules, no styled-components, no Tailwind

### Primer React usage
Import components from `@primer/react`, icons from `@primer/octicons-react`:
```jsx
import { Box, Button, UnderlineNav } from "@primer/react";
import { RepoIcon } from "@primer/octicons-react";
```
Use the `sx` prop for one-off style overrides on Primer components (they accept a subset of CSS-in-JS).
