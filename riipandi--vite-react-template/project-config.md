---
trigger: always_on
description: Starter template for a React 19 + Vite 8 + TypeScript SPA using TanStack Router/Query/Form/Store, StyleX, and Base UI.
---

# AGENTS.md

Starter template for a React 19 + Vite 8 + TypeScript SPA using TanStack Router/Query/Form/Store, StyleX, and Base UI.
You are helping maintain this template for its author.

## Tooling

- React 19 (**React Compiler enabled**) + Vite 8 + TypeScript; Node >= 24.12 and pnpm 12 (`engines`/`packageManager`).
- Styling: StyleX everywhere (+ `@stylexjs/atoms` for one-off utilities), Base UI headless primitives, Keyline Icons, ofetch for HTTP.
- Lint/format: Oxc tools (`oxlint`, `oxfmt`) — no ESLint/Prettier. Storybook 10 (`@storybook/tanstack-react`) with a11y, docs, links, mcp, vitest, performance addons.

## Commands (pnpm)

- `pnpm dev` — prebuild typecheck + dev server, port 3000 (strict; fails if busy).
- `pnpm build` — `tsr generate && tsc -b` + app build + Storybook build → `.output/build/` and `.output/build/storybook/` (what Netlify deploys, see `netlify.toml`).
- `pnpm start` — serve `.output/build/` via `vite preview` (build first or you serve stale output).
- `pnpm test` — unit tests (Vitest, project `unit`); `pnpm test:storybook` — Storybook interaction tests (real Chromium via Playwright); `pnpm test:coverage` — with coverage (thresholds in `vitest.config.ts`); `pnpm test:ui` — open the tokened URL printed in the terminal.
- `pnpm lint` / `pnpm format` — write fixes in place; `pnpm check` verifies formatting only.
- `pnpm typecheck` — `tsc -b --noEmit`.
- `pnpm storybook` — dev server, port 6006. `pnpm knip` — unused exports/files check. `pnpm pre-commit` — full lefthook suite (format, lint, typecheck, unit tests).

## Architecture

- Source root is `app/`, imported via the `#/*` alias (`#/components/...`); relative imports stay within a directory.
  - `routes/` — file-based TanStack Router routes (groups `(app)`, `(auth)`). The `(app)` guard redirects anonymous visitors to `/login?return_to=…`; `(auth)` sends already-signed-in visitors to `return_to` or `/overview`.
  - `components/` — shared components (`base/`, `charts/`, `extra/`, `icons/`, `theme/`).
  - `styles/` — StyleX design tokens and styling (`core/`, `element/`, `pages/`).
  - `schemas/` — Zod schemas (shared with TanStack Form validation); `hooks/` — shared React hooks; `libraries/` — `api-client.ts` (QueryClient + ofetch), `app.store.ts` (TanStack Store), `guard/` (cookie-session auth via a Comlink token worker), `utils/`.
  - `tests/` — unit tests for non-component code only (pages in `tests/page/`, guard in `tests/guard/`).
- `app/routes.gen.ts` and `.tanstack/` are generated (`tsr generate`; also runs inside `pnpm build`) — never hand-edit; implement routes in `app/routes/` and regenerate.

### Component structure & testing

- Folder layout in `app/components/<area>/<name>/`: `<name>.component.tsx`, `<name>.stylex.ts` (exports `<name>Styles`), `<name>.stories.tsx` (colocated), `index.ts`.
- **No unit tests for components.** Stories are the component's tests — run via `pnpm test:storybook` (browser mode, a11y checks via `@storybook/addon-a11y`). Add stories for every meaningful UI state. Unit tests are for pages, libraries, and utilities only.

## React 19 & Performance

- **React Compiler is active** in the app build: `react({ compiler: true })` with `oxc-transform-react` (native oxc path — do not switch to the Babel/`babel-plugin-react-compiler` route). Defaults: `compilationMode: 'infer'`, `panicThreshold: 'none'` — components that violate the Rules of React are skipped, never broken. Storybook intentionally compiles WITHOUT the compiler (uncompiled reference).
- **Purity is a hard rule.** Render functions and state updater functions must be side-effect free — no storage writes, DOM mutations, or fetches. StrictMode double-invokes updaters and the compiler assumes purity. Move side effects into effects/handlers (see the `setTheme` persistence effect in `theme.tsx`).
- Don't add `useMemo`/`useCallback`/`memo` by default — the compiler memoizes. Keep existing manual memo (removal is gradual — profile first). Escape hatch: the `"use no memo"` directive.
- `ref` is a plain prop — no `forwardRef` in React 19.
- Use `useId()` for element ids referenced in `url(#…)` or the DOM (see `icons/`) — sanitize before interpolation.
- Subscribe narrowly: TanStack Store `useSelector(store, narrowSelector)`; prefer scoped hooks (e.g. `useAuthUser`) over whole-store reads to avoid redundant re-renders.
- `use(promise)` requires a cached promise (e.g. a deferred router-loader result) plus a Suspense boundary; never call it inside try/catch; never read `promise.status` manually.
- TanStack Query v5.9x+: use `queryClient.query()` / `infiniteQuery()` (with `staleTime: 'static'` for fetch-once semantics) — `fetchQuery`/`ensureQueryData`/`prefetchQuery` are deprecated.
- Router: `defaultPreload: 'intent'` and `scrollRestoration` are set; loaders may return unawaited promises consumed via `use()` (non-blocking deferral).
- `ThemeProvider` runs with `disableTransitionOnChange` — theme swaps snap in one frame (mixed-speed color transitions read as a flash). Don't remove it.

## Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [riipandi/vite-react-template](https://github.com/riipandi/vite-react-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
