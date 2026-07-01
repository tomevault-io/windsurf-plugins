---
trigger: always_on
description: Custom Apollo Link that parses and serializes custom GraphQL scalars on the client. Supports Apollo Client v3 and v4. Also exposes `reviveScalarsInCache` for rehydrating scalar values in a JSON-persisted Apollo cache.
---

# apollo-link-scalars — agent notes

## What this repo ships

Custom Apollo Link that parses and serializes custom GraphQL scalars on the client. Supports Apollo Client v3 and v4. Also exposes `reviveScalarsInCache` for rehydrating scalar values in a JSON-persisted Apollo cache.

Entry: `src/index.ts`. Main runtime type is `ScalarApolloLink` in `src/lib/link.ts`; helper is `withScalars(...)`.

## Repo layout (pnpm workspace)

- `src/` — library source (TypeScript)
- `build/main/` — CJS build output (ships as `main` in package.json)
- `build/module/` — ESM build output (ships as `module`)
- `test-apps/` — pnpm workspace packages that consume `apollo-link-scalars` via `workspace:*`
  - `graphql-test-server` — Apollo Server (express + graphql-ws) on port 5178. Mirrors the rickandmortyapi.com schema for reads and adds mutations + subscriptions on top. State partitioned by `X-Run-Id` header; missing header routes to a shared read-only seed. Used by the integration spec and all Vite-based test-apps.
  - `apollo-v3-react` — Vite + React, Apollo v3, port 5173. Hits the local test server for `character(id)`.
  - `apollo-v4-react` — Vite + React, Apollo v4, port 5174. Hits the local test server for `character(id)`.
  - `apollo-v4-persisted-cache` — Vite + React, Apollo v4 with cache persistence, port 5175. Hits the local test server for `character(id)`.
  - `apollo-v4-next-ssr` — Next.js 16, Apollo v4 SSR, port 5176.
  - `apollo-v4-issue-1041` — Vite + React, Apollo v4, port 5177. Isolated repro for issue #1041 (cache-first with custom-serialized variables). Hosts its own minimal `events(at: DateTime)` GraphQL schema via vite `configureServer` middleware.
  - `apollo-v4-issue-1565` — Vite + React, Apollo v4, port 5179. Isolated repro for issue #1565 (`withScalars` link no-ops in `vite build` output because graphql-js class names get minified). Runs against `vite build && vite preview` (NOT `vite dev`) since the bug only reproduces in the minified production bundle. Hosts its own `Date` scalar schema via vite `configureServer` AND `configurePreviewServer` middleware.
  - `apollo-v4-issue-bigint` — Vite + React, Apollo v4, port 5180. Repro for the post-5.0.1 BigInt regression: a cache-first query and a mutation, both with a `BigInt` variable. Exercises `withScalars({ ensureSerializableVariables: true })`. Hosts its own `BigInt` scalar schema via vite `configureServer` middleware.
- `playwright.config.ts` (root) — boots the test server first, then the seven app servers (six via `vite dev`, plus `apollo-v4-issue-1565` via `vite build && vite preview`)

## CRITICAL build gotcha — test apps consume `build/`, not `src/`

`workspace:*` resolves through package.json `main: "build/main/index.js"` / `module: "build/module/index.js"`. Vite, Next, and anything else in `test-apps/` ALWAYS see the compiled artifact.

Consequence: editing `src/` alone does not affect `test-apps/` tests until you run `pnpm run build`. Unit tests (`vitest run`) compile from `src/` directly and are unaffected.

Flow for RED/GREEN reproductions at the e2e layer:

1. Edit `src/lib/...`
2. `pnpm run build`
3. Kill lingering Vite/Next dev servers: `lsof -ti:5173,5174,5175,5176,5177 | xargs -r kill -9` (Playwright's `reuseExistingServer: !CI` means stale servers stick across runs; Vite caches transformed deps even after rebuild)
4. `pnpm exec playwright test --project=apollo-v4-issue-1041` (or the relevant project)

For unit repros, `npx vitest run path/to/spec.ts` is enough — no build needed.

## Test runner

Unit: **vitest** (not jest). `pnpm test:unit` → `vitest run --coverage`. Specs live in `src/**/*.spec.ts`.

Integration: `pnpm test:integration` (gated by `RUN_INTEGRATION=1`, specs in `src/__tests__/integration`).

E2E: Playwright root config. `pnpm e2e` = build then `playwright test`. Single worker, 2 retries, Chromium.

Full matrix: `pnpm test:matrix` runs against Apollo v3 and v4 sequentially by swapping the installed `@apollo/client` version.

## Common scripts

- `pnpm run build` — clean then `tsc` for main + module outputs in parallel
- `pnpm test:lib` — build + eslint + prettier check + unit + dep cycle check (dpdm) + dead code (knip) + type-check
- `pnpm test:apps` — type-check + lint + build across all test-apps
- `pnpm test:full` — `test` + `test:integration`
- `pnpm exec playwright test --project=<name>` — targeted e2e. Projects: `apollo-v3-react`, `apollo-v4-react`, `apollo-v4-persisted-cache`, `apollo-v4-next-ssr`, `apollo-v4-issue-1041`, `apollo-v4-issue-1565`, `apollo-v4-issue-bigint`.

## Issue #1041 — cache-first with custom-serialized variables

**Bug:** `ScalarApolloLink.cleanVariables` mutated `operation.variables` in place, replacing Date-like instances with their custom-serialized string. Apollo internally uses that same variables object for cache-key identity and for cache writes, so a roundtrip `A → B → A` (fresh Date each time) would:
- Write cache keyed on the custom serialized string (e.g. `"2023-08-19T00:00:00.000+00:00"`)
- Miss on the third read because fresh Date stringifies via `toJSON` to ISO-Z (`"2023-08-19T00:00:00.000Z"`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eturino/apollo-link-scalars](https://github.com/eturino/apollo-link-scalars) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
