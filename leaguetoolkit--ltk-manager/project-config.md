---
trigger: always_on
description: Conventions for everything under `src/`. Repo-wide guidance lives in the root `CLAUDE.md`.
---

# Frontend (React + TypeScript) - `src/`

Conventions for everything under `src/`. Repo-wide guidance lives in the root `CLAUDE.md`.

## JSX Conditional Rendering

**Avoid ternary operators in JSX.** Use early returns or `{condition && <Component />}` instead.

```tsx
// Good - early return
if (isLoading) return <LoadingState />;
if (error) return <ErrorState error={error} />;
return <Content />;

// Good - single-line conditional
{
  hasItems && <ItemList items={items} />;
}

// Bad - ternary in JSX
{
  isLoading ? <LoadingState /> : error ? <ErrorState /> : <Content />;
}
```

## Import Conventions

**Always import from barrel exports, never from subdirectories.** This keeps import paths stable and encapsulates internal structure.

- **Global components:** import from `@/components`, not `@/components/Button`, `@/components/Toast`, etc.
- **Modules:** import from `@/modules/{module}`, not `@/modules/{module}/components` or `@/modules/{module}/api`.

```ts
// Good
import { Button, IconButton, useToast } from "@/components";
import { ModCard, useInstalledMods } from "@/modules/library";

// Bad - reaches into internals
import { Button } from "@/components/Button";
import { useToast } from "@/components/Toast";
import { ModCard } from "@/modules/library/components";
```

## State Consumption - Hooks Over Prop Drilling

**Consume global state (hooks, queries, stores) directly in the component that needs it.** Do not drill Zustand state, TanStack Query data, or mutation callbacks through intermediate components as props.

- Patcher status → call `usePatcherStatus()` in the component that checks it
- Mod toggle/uninstall → call `useToggleMod()` / `useUninstallMod()` in `ModCard`, not passed from a parent
- Folder toggle → call `useFolderToggle()` in `FolderRow`/`FolderCard`, not received as a prop

TanStack Query deduplicates identical queries, so multiple components calling the same hook is efficient and correct.

**Exception:** Props are appropriate for coordinating parent-owned UI state (e.g., `onViewDetails` that opens a sibling dialog, `onReorder` where reorder target varies by context).

## Tauri Event Listening

For backend-to-frontend events (e.g., overlay progress), use `listen<T>()` from `@tauri-apps/api/event` in a `useEffect` with cleanup via `unlisten()`. See `modules/patcher/api/useOverlayProgress.ts` for the pattern.

## Routing

TanStack Router with file-based routing in `src/routes/`. Route tree is auto-generated in `routeTree.gen.ts`. The root route (`__root.tsx`) checks setup status and redirects to `/settings` on first run.

## Component Library (`src/components/`)

**ALWAYS use reusable components from `@/components` instead of native HTML or raw base-ui imports.** Module code should never import from `@base-ui-components/react` directly - all base-ui primitives must be wrapped in `src/components/` first. See `src/components/index.ts` for what is already wrapped.

When adding a new base-ui component:

1. Create wrapper in `src/components/NewComponent.tsx`
2. Export from `src/components/index.ts`
3. Import in modules via `@/components`, never from `@base-ui-components/react` directly

## Form System (`src/lib/form/`)

Uses `@tanstack/react-form` with Zod validation via `useAppForm()`. Field components are pre-registered on `form.AppField` / `form.AppForm` and integrate with the wrapped `@/components` primitives.

## Dependency Constraints

- `zustand` - client-side state only. Never use it for server state - that is TanStack Query's job.
- `framer-motion` - Layout animations for DnD (`AnimatePresence` on `DragDropOverlay` only). Tree-shake to ≤30KB gzipped.

## Icons

All icons come from `@phosphor-icons/react`, imported by PascalCase name. Standard spinner is
`<SpinnerGap className="animate-spin" />`.

`lucide-react` is still installed because most of the app still imports it, and it stays until those
call sites are converted. **Write no new lucide imports** - a file being touched for something else
is a fine moment to convert the icons in it.

Phosphor names things by shape rather than by role, so the lucide name is rarely the phosphor name:
`ChevronDown` is `CaretDown`, `Search` is `MagnifyingGlass`, `Settings` is `Gear`, `Trash2` is
`Trash`, `Loader2` is `SpinnerGap`. Look the name up rather than guessing.

Phosphor's `regular` weight is lighter than lucide's 2px stroke, so a converted icon reads thinner
beside one that has not been converted yet. Pass `weight="bold"` where an icon carries an action -
buttons, toolbar controls - and leave `regular` for decorative and section-header icons.

Riot's own marks are the exception, since neither icon set carries them. They live as inline-SVG
components in `src/components/icons/`, lifted from the League and Riot Client asset sets:
`LeagueIcon`, `RiotIcon`, `TftIcon`, and the cosmetics family `MaskIcon` / `ThreeMasksIcon` /
`EvolutionIcon`. That folder has its own barrel, re-exported by `src/components/index.ts` - call
sites still import from `@/components`.

Keep the path data untouched, and change only what stops it behaving like an icon: swap the client's
hardcoded fill (League gold `#C89B3C`, parchment `#F0E6D2`) for `currentColor`, drop any wrapping

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeagueToolkit/ltk-manager](https://github.com/LeagueToolkit/ltk-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
