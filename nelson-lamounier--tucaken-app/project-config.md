---
trigger: always_on
description: Authoritative guide for Codex when working in this repo. Read fully before editing.
---

# AGENTS.md — tucaken-app

Authoritative guide for Codex when working in this repo. Read fully before editing.

## Stack

- **Runtime**: TanStack Start (SSR) + TanStack Router + TanStack Query + TanStack Form
- **React**: 19 (Server Components aware)
- **Build**: Vite 8, esbuild for server bundle
- **Styling**: Tailwind CSS v4 via `@tailwindcss/vite` — config lives in `src/styles.css` `@theme` block (no `tailwind.config.*`)
- **State**: Zustand (client), TanStack Query (server cache)
- **Validation**: Zod
- **Animation**: `motion` (Motion for React) — import from `motion/react`, never `framer-motion`
- **Auth**: AWS Cognito + `jose`
- **Payments**: Stripe (`@stripe/*`, server `stripe`)
- **Observability**: OpenTelemetry, Pyroscope, Grafana Faro, Pino
- **Tests**: Vitest

Workspace: yarn workspaces. Root + `admin-api/`.

## Package manager — Yarn 4 only

`packageManager: yarn@4.12.0`. Never use `npm`/`pnpm`/`npx`.

- Install: `yarn install`
- Add dep: `yarn add <pkg>` (root) or `yarn workspace admin-api add <pkg>`
- Add dev dep: `yarn add -D <pkg>`
- Run script: `yarn <script>` (not `npm run`)
- Execute binary: `yarn dlx <cmd>` instead of `npx`
- Lockfile `yarn.lock` is committed. Never delete/regenerate without reason.

## Scripts

| Script | Purpose |
|---|---|
| `yarn dev` | Vite dev on port 5001 |
| `yarn build` | Vite build + server bundle |
| `yarn preview` | Preview production build |
| `yarn lint` | ESLint |
| `yarn typecheck` | `tsc --noEmit` |
| `yarn test` | Vitest run |

Before claiming a task done: `yarn typecheck && yarn lint && yarn test`.

## Repo layout

```
src/
  app/                 # TanStack Start flat-file routes (filename dots = segments)
    __root.tsx
    _dashboard.*.tsx   # routes under dashboard layout group
  features/<domain>/   # feature-sliced — primary location for new code
    components/
    hooks/
    api/ or server/
    types.ts
    store.ts           # zustand if needed
  components/
    ui/                # shared primitives (Button, Field, etc.)
    layouts/
    resume/
  contexts/            # cross-feature React contexts only
  hooks/               # cross-feature hooks only
  lib/                 # framework-agnostic utilities, clients, observability
  server/              # server-only code, patches, handlers
  styles.css           # Tailwind v4 @theme tokens
  router.tsx
  routeTree.gen.ts     # generated — never edit
admin-api/             # workspace (separate package)
```

### Where new code goes

- Belongs to a domain (auth, billing, profile, resumes, …) → `src/features/<domain>/`
- Generic primitive used in 2+ features → `src/components/ui/`
- Server-only (Node APIs, secrets, SDKs) → `src/server/` or `src/lib/<area>/server.ts`
- Route file → `src/app/` only. Never put business logic in route files; import from features.

## TanStack — modern patterns

- **Routing**: file-based in `src/app/`. Legacy routes use flat-file convention (dots = segments, e.g. `_dashboard.applications.$slug.tsx`). `_dashboard` = layout group. `$param` = dynamic. `routeTree.gen.ts` is generated — **never** hand-edit.
- **Routing migration (incremental, mandatory for new work)**: repo is migrating flat-file → directory-based. TanStack Router supports both simultaneously; mixing during transition is intended.
  - **New routes**: create directory-based only. `src/app/<segment>/<sub>/route.tsx` (and `index.tsx` for the segment's own page, `$param/route.tsx` for dynamics).
  - **Touching an existing flat route**: if the change is non-trivial (new sibling, rename, splitting logic), migrate that route's whole prefix group to directory form in the same PR. Update all imports; run `yarn typecheck`.
  - **Migration order priority** (high fan-out first): `_dashboard.applications.*` → `_dashboard.resumes.*` → `_dashboard.settings.*` → remaining `_dashboard.*` leaves → top-level routes (`checkout.*`, `sign-in.*`, `github.*`, `articles.preview.*`).
  - **Mirror feature slices**: directory tree under `src/app/_dashboard/<domain>/` should match `src/features/<domain>/`. Route file stays thin — import logic from the feature.
  - **Colocation rule**: route-private helpers go next to `route.tsx` as `-loader.ts`, `-components/`, etc. Shared logic stays in `src/features/<domain>/`.
  - **No new flat-file routes.** Only exception: a one-off leaf with no siblings and no expected growth — document why in PR description.
  - Do not rewrite untouched flat routes en masse; migration is opportunistic, driven by real work.
- **Data**: prefer `createServerFn` for server logic + `queryOptions` for the client. SSR hydration via `@tanstack/react-start` + `@tanstack/react-router-ssr-query`.
- **Loaders**: route `loader` should call `queryClient.ensureQueryData(queryOptions)` so the same query is reused client-side. Avoid raw `fetch` in components.
- **Forms**: `@tanstack/react-form` + `zod-form-adapter`. Schemas live with the feature.
- **Devtools**: `@tanstack/react-router-devtools`, `@tanstack/react-query-devtools` only mounted in dev.
- **Type safety**: rely on generated route types; never cast `as any` to silence the router.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nelson-Lamounier/tucaken-app](https://github.com/Nelson-Lamounier/tucaken-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
