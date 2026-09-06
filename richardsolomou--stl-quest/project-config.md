---
trigger: always_on
description: Read [CONTRIBUTING.md](CONTRIBUTING.md) first: it defines the layout (`src/core` isomorphic domain → `src/adapters`/`src/db` → `src/server`/`src/client`/`src/routes`), database rules, and release-note policy. This file adds the operational detail that isn't obvious from reading it.
---

# STL Quest — Agent Guide

Read [CONTRIBUTING.md](CONTRIBUTING.md) first: it defines the layout (`src/core` isomorphic domain → `src/adapters`/`src/db` → `src/server`/`src/client`/`src/routes`), database rules, and release-note policy. This file adds the operational detail that isn't obvious from reading it.

## Commands

- `just check` — the full local gate (format, lint, `db:check`, `catalog:check`, build, typecheck, unit tests, backup CLI smoke). The build runs **before** typecheck because it generates `src/routeTree.gen.ts`; on a fresh clone, typecheck fails until you build.
- Dev server: `just dev` starts Vite and the realtime service together, then stops realtime when Vite exits. Use `just realtime` only to debug that service separately.
- Unit tests: `just test`. Vitest runs with `fileParallelism: false` because of the `globalThis.__stlquest` app singleton and shared SQLite state — don't assume isolation across test files.
- E2E: `just e2e` builds and tests the production server; `just e2e-run` reruns the current build, and `just e2e-trace` records a local trace (see the `extending-e2e` skill). Install Chromium once with `just e2e-install`; set `PLAYWRIGHT_DEV_SERVER=1` only when debugging against Vite.
- When optimizing E2E runtime, measure the default local command separately from CI and improve both paths.
- Lint/format is oxlint + oxfmt (`just lint`, `just format`), not ESLint/Prettier. Warnings are denied in CI.
- Toolchain: Node 24.x only (`engines` pins `>=24 <25`), pnpm 11.15.0 via the `packageManager` field, and just 1.58.0.

## Load-bearing rules

- **Server functions** (`src/server/fns.ts`): wrap reads in `rpc()` and mutations in `mutationRpc()` (or the narrower `workspaceMutation()`) — thrown `Response` objects otherwise reach the client as a _successful_ result, and mutations need the origin check before any state access. CSRF protection is enforced by these wrappers, not middleware. See the `adding-server-functions` skill.
- **Authorization lives in server functions**, not routes. Route `beforeLoad`/`useEffect` redirects are UX only.
- **Workspace isolation is absolute**: every tenant table carries `workspace_id` with a composite FK to its parent; every `DrizzleRepository` (`src/db/repository.ts`) method filters via the scoped repository (`scoped(workspaceId)`). New tenant tables and queries must follow suit — there is no bypass path.
- **Client queries**: `queryOptions` factories live in `src/client/queries.ts`, never inline. Workspace-scoped query keys must include `workspaceSlug` or data leaks across workspace switches. Invalidation is blanket via the workspace realtime channel — no bespoke invalidation needed.
- **`AppEvent`** (`src/core/types.ts`) is a closed union treated as a public API: additions are fine, renames/removals are breaking. Server-side state changes publish one, and mutations go through `STLQuestService`, not the repository.
- **Settings, not env vars**: product configuration goes in the `settings` (workspace) or `deployment_settings` (global) tables. Env vars are reserved for filesystem paths, operational controls, recovery, and managed-deployment overrides. See the `adding-a-setting` skill.
- **Telemetry stays useful and anonymous**: when adding or changing a meaningful workflow, consider whether a success event would answer a concrete product-health question. Capture only the minimum useful properties after success, prefer server-side capture in `STLQuestService`, and update `docs/telemetry.md` plus tests. Random internal IDs, roles, categorical state, counts, and automatic in-app navigation URLs are allowed; never send names, emails, user-provided content, filenames, user-provided URLs, storage endpoints, credentials, or secrets. Do not add events merely for coverage.
- **CSP is a hardcoded string in `vite.config.ts`** (under `nitro.routeRules`). Any new external image/script/connect source (OAuth avatar CDNs, telemetry hosts) requires editing it — easy to miss.
- **`AssetStore` has a behavioral contract**: `src/adapters/storeContract.test.ts` runs the same suite against the local and S3 stores (S3 gated on `MINIO_TEST_*` env vars) — semantic changes must extend it so both stay equivalent. Crash recovery replays the operation journal (`STLQuestService.resumeOperation`); a new operation kind must extend that state machine and its recovery tests.
- **Asset migrations are permanent**: stored-model or provider-folder changes use a new numbered file in `src/server/assetMigrations/` and append it to the registry. Never edit, reorder, rename, or remove a released asset migration; skipped releases must run every missing migration in order.
- **The asset worker is bundled separately**: `pnpm build` runs `src/server/assets/worker.ts` through its own esbuild pass (not the Vite/Nitro bundle) to `assets-worker.mjs`. New imports there must survive standalone bundling; tests run the queue inline (`process.env.VITEST`), so worker-only breakage won't show in unit tests.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [richardsolomou/stl.quest](https://github.com/richardsolomou/stl.quest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
