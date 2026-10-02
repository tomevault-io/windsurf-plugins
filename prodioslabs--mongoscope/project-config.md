---
trigger: always_on
description: Put code next to its only consumer; promote on a second real consumer. Domain stays outside `app/`; React/OpenTUI stays inside `app/`.
---

# Agent guidelines

## Folder placement

Put code next to its only consumer; promote on a second real consumer. Domain stays outside `app/`; React/OpenTUI stays inside `app/`.

| Kind of code                                                 | Where it goes                                                                                                                                                 | Must not                        |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- |
| Domain logic (pure, no OpenTUI)                              | Top-level `src/<domain>/` — today: `connections`, `parser`, `query-patterns`, `indexes`, `live-ops`, `live-connection`, `log-tail`, `profiler`, `replication` | Import from `app/`              |
| Domain types                                                 | Colocated `types.ts` in that domain folder (e.g. `src/connections/types.ts`)                                                                                  | Central `lib/types.ts` dump     |
| Cross-domain pure utils                                      | `src/lib/` by concern                                                                                                                                         | Import domain modules           |
| OpenTUI shell, screens, stores, shortcuts, theme, app config | `src/app/`                                                                                                                                                    | Own domain business rules       |
| TUI-only helpers                                             | `src/app/lib/`                                                                                                                                                | Live in `src/lib/`              |
| React Query / UI data hooks                                  | `src/app/queries/`                                                                                                                                            | Sit at top-level `src/queries/` |
| Feature screens + feature-private presentation               | `src/app/components/<feature>/<name>/`                                                                                                                        | Premature extract to shared     |
| Shared widgets / primitives (2+ features)                    | `src/app/components/<name>/` or `ui/`                                                                                                                         | Feature-private helpers         |
| Process entry (yargs)                                        | `src/cli/`                                                                                                                                                    | App UI                          |
| Tests                                                        | Colocated `*.test.ts` next to source                                                                                                                          | Separate `tests/` tree          |

Reuse ladder:

```text
Used by one feature screen?  → app/components/<feature>/
Used by 2+ features?         → app/components/ or app/components/ui/
Used by domain + app, pure?  → src/lib/
Domain-only helper?          → that domain folder
```

## Component folders

Every non-`ui/` component lives in its own kebab-case folder with an `index.ts` barrel (same rule as coa-erp-portal shared components):

```text
src/app/components/<component-name>/
├── index.ts                 # export { ComponentName } from './component-name'
└── component-name.tsx
```

Feature screens follow the same pattern under a feature prefix:

```text
src/app/components/connections/connections-dialog/
├── index.ts
└── connections-dialog.tsx
```

`src/app/components/ui/` stays flat — import primitives by file (`ui/dialog`), not folder barrels.

## Component props

Define a named props type (expanded, one field per line) and destructure it in the component signature. Rename when a prop collides with a local binding.

```tsx
// ❌ BAD
export function ThemeProvider(props: { mode?: ThemeMode; theme?: string; children?: ReactNode }) {
  return <>{props.children}</>
}

// ✅ GOOD
type ThemeProviderProps = {
  mode?: ThemeMode
  theme?: string
  children?: ReactNode
}

export function ThemeProvider({ mode, theme: themeName, children }: ThemeProviderProps) {
  return <>{children}</>
}
```

## React effects

Always pass a **named function** to `useEffect` and `useLayoutEffect`. The name must convey the effect’s role (what it synchronizes, subscribes to, or applies).

```tsx
// ❌ BAD — anonymous arrow hides intent in stacks and reviews
useEffect(() => {
  renderer.setBackgroundColor(theme.background)
}, [renderer, theme.background])

// ✅ GOOD — name states the effect’s job
useEffect(
  function applyThemeBackground() {
    renderer.setBackgroundColor(theme.background)
  },
  [renderer, theme.background],
)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prodioslabs/mongoscope](https://github.com/prodioslabs/mongoscope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
