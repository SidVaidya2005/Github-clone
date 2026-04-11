# Frontend Patterns

Patterns observed across the codebase. Follow these when adding or modifying frontend code.

## Auth-gated data fetching

Always read `userId` from `localStorage` inside `useEffect` — not from context — because context hydration is async and may not be ready when the effect runs:

```jsx
useEffect(() => {
  const userId = localStorage.getItem("userId");
  if (!userId) return;

  const fetchData = async () => {
    try {
      const res = await axios.get(`http://localhost:3002/some-route/${userId}`);
      setData(res.data);
    } catch (err) {
      console.error("...", err);
    }
  };

  fetchData();
}, []);
```

## Page layout pattern

Every page component renders `<Navbar />` as the first child:

```jsx
return (
  <>
    <Navbar />
    <section id="page-name">
      {/* content */}
    </section>
  </>
);
```

## Logout

Logout always does three things in order:
1. `localStorage.removeItem("token")`
2. `localStorage.removeItem("userId")`
3. `setCurrentUser(null)` then `window.location.href = "/auth"`

Never use `navigate()` for logout — the full page reload is intentional to clear any in-memory state.

## Controlled inputs

All form inputs are controlled with inline `onChange`:

```jsx
<input
  value={email}
  onChange={(e) => setEmail(e.target.value)}
/>
```

## Loading states

Use a `loading` boolean state for async form submissions. Disable the submit button while loading:

```jsx
const [loading, setLoading] = useState(false);
// in handler:
setLoading(true);
// ... await ...
setLoading(false);

<Button disabled={loading}>{loading ? "Loading..." : "Submit"}</Button>
```

## Primer `sx` overrides

When Primer component styles conflict with the project's dark theme, override via `sx`:

```jsx
<UnderlineNav.Item
  sx={{
    backgroundColor: "transparent",
    color: "white",
    "&:hover": { color: "white" },
  }}
>
```

For heavier overrides (width, font, full background), use CSS class + `!important` as done in `auth.css`.

## Routing

There is no `<Switch>` — the app uses `useRoutes()` from react-router-dom v6. To add a route, add an entry to the array in `Routes.jsx` and ensure the component is inside `<AuthProvider>`.

## Missing route: `/create`

`Navbar.jsx` links to `/create` ("Create a Repository") but no route or component exists for it. This will silently render nothing. Implement `CreateRepo` component and add the route before linking to it.
