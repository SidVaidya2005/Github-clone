# Design System

All visual tokens live in `frontend/src/index.css` as CSS custom properties on `:root`. Always use these variables — never hardcode colors, radii, or shadows.

## Color tokens

### Backgrounds
| Token | Value | Use |
|-------|-------|-----|
| `--bg-base` | `#090e1a` | Page background |
| `--bg-elevated` | `#0f1623` | Elevated surfaces |
| `--bg-card` | `#131c2e` | Cards, form boxes |
| `--bg-input` | `#0b1120` | Input fields |
| `--bg-hover` | `rgba(255,255,255,0.04)` | Hover overlays |

### Borders
| Token | Value | Use |
|-------|-------|-----|
| `--border` | `rgba(255,255,255,0.07)` | Default border |
| `--border-hover` | `rgba(255,255,255,0.14)` | Hover state |
| `--border-focus` | `#3ecfb5` | Focus ring (same as accent) |

### Text
| Token | Value | Use |
|-------|-------|-----|
| `--text-primary` | `#e2eaf6` | Body text |
| `--text-secondary` | `#8896a8` | Labels, captions |
| `--text-muted` | `#4d5c70` | Placeholders, disabled |

### Accent (teal)
| Token | Value |
|-------|-------|
| `--accent` | `#3ecfb5` |
| `--accent-dim` | `rgba(62,207,181,0.12)` |
| `--accent-hover` | `#5cdbc9` |
| `--accent-glow` | `rgba(62,207,181,0.25)` |

### Status
| Token | Value |
|-------|-------|
| `--green` | `#3dd68c` |
| `--green-dim` | `rgba(61,214,140,0.12)` |
| `--danger` | `#f85149` |
| `--warning` | `#d29922` |

## Typography

Fonts are loaded from Google Fonts:
- **Sans:** `DM Sans` (300, 400, 500, 600) — fallback: system-ui
- **Mono:** `DM Mono` (400, 500) — fallback: Fira Code, Cascadia Code

| Token | Value |
|-------|-------|
| `--font-sans` | `'DM Sans', -apple-system, BlinkMacSystemFont, sans-serif` |
| `--font-mono` | `'DM Mono', 'Fira Code', 'Cascadia Code', monospace` |

Heading `letter-spacing`: `-0.02em` (set globally). Label `letter-spacing`: `0.01em`.

## Spacing & Radii

| Token | Value | Use |
|-------|-------|-----|
| `--radius-sm` | `6px` | Inputs, small buttons |
| `--radius-md` | `10px` | Secondary cards |
| `--radius-lg` | `14px` | Main cards (login box) |
| `--radius-xl` | `20px` | Large containers |

## Shadows

| Token | Value |
|-------|-------|
| `--shadow-sm` | `0 1px 4px rgba(0,0,0,0.5)` |
| `--shadow-md` | `0 4px 20px rgba(0,0,0,0.55)` |
| `--shadow-card` | `0 0 0 1px var(--border), 0 8px 32px rgba(0,0,0,0.45)` |
| `--shadow-focus` | `0 0 0 3px var(--accent-glow)` |

## Transitions

```css
--transition: 150ms cubic-bezier(0.4, 0, 0.2, 1)
```
Always use `transition: <property> var(--transition)` for hover/focus states.

## Primer overrides

`@primer/react` components render with their own dark-mode styles. When overriding Primer inside custom CSS, use `!important` sparingly and only on properties that conflict with Primer's inline styles. Examples in `auth.css`:
- Primer `PageHeader` background/border reset
- Primer `Button` width, height, background, border, font

## Scrollbar

Thin 5px scrollbar styled globally in `index.css`. Do not add per-component scrollbar styles.
