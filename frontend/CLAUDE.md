# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install
npm run dev      # Vite dev server (http://localhost:5173)
npm run build    # Production build
npm run lint     # ESLint (max-warnings 0)
npm run test     # Vitest
```

## Environment

All API calls are hardcoded to `http://localhost:3002`. The backend must run on port 3002 (set `PORT=3002` in backend `.env`).

## Architecture

### Auth flow
`main.jsx` wraps the app in `<AuthProvider>` — any new top-level component that needs auth state must be inside this provider.
`authContext.jsx` provides `currentUser` (userId string) and `setCurrentUser` via React Context. The value is persisted to / hydrated from `localStorage` key `"userId"`. `Routes.jsx` handles redirect logic on mount — unauthenticated users go to `/auth`, already-authenticated users are bounced away from `/auth` to `/`.

### Routes
| Path | Component |
|------|-----------|
| `/` | `Dashboard` |
| `/auth` | `Login` |
| `/signup` | `Signup` |
| `/profile` | `Profile` |

### Key dependencies
- **@primer/react** — GitHub Primer design system components
- **@uiw/react-heat-map** — contribution heatmap in Profile
- **axios** — imported but not used in all components; `Dashboard.jsx` uses native `fetch()`. HTTP client usage is inconsistent across components.
- **react-router-dom v6** — `useRoutes` + `useNavigate` (no `<Switch>`)

### Testing
Vitest + `@testing-library/react`. Test files colocate with components or live under `src/`. Run a single test file: `npx vitest run <path>`.
