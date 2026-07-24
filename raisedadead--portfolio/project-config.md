---
trigger: always_on
description: Project notes — non-obvious only. Commands, deps, and tsconfig are discoverable from `package.json` / `tsconfig.json`; not duplicated here.
---

# CLAUDE.md

Project notes — non-obvious only. Commands, deps, and tsconfig are discoverable from `package.json` / `tsconfig.json`; not duplicated here.

## Stack

Astro 6 (SSR, `@astrojs/cloudflare` 13 adapter) + React 19 islands + Tailwind 4. Markdown sourced from R2 at build time; freeCodeCamp RSS as a second collection. Sentry client + server. pnpm enforced via `packageManager`.

## pnpm 11 — non-obvious gotcha

`postinstall` is `wrangler types` — a **direct** call, not `turbo cf-typegen`. This is load-bearing: routing `postinstall` through turbo spawns a nested `pnpm run cf-typegen` mid-install, and pnpm 11's default re-verifies deps on every `pnpm run`, auto-installs when node_modules looks stale, and recurses infinitely (CI dies with exit 137 OOM / 143 SIGTERM; turbo cache hits masked it locally). A direct binary call has no nested `pnpm run`, so the loop cannot form — critical inside Cloudflare Workers Builds, whose sandbox has no turbo remote-cache token.

`verifyDepsBeforeRun: false` in `pnpm-workspace.yaml` is retained as defense-in-depth against the same auto-install behavior on any other `pnpm run` during install. Native build scripts are approved via `allowBuilds` (map form — the v10 `onlyBuiltDependencies`/`ignoredBuiltDependencies` keys were removed in pnpm 11 and are silently ignored).

## Turbo vs Cloudflare Workers Builds — non-obvious gotcha

CF Workers Builds runs **build and deploy as two separate commands** in the same container, so the deploy command must not re-build. Script tiers:

- Top-level (`build`, `deploy`, `lint`, `test`, …) → `turbo do:*`. Local + GitHub Actions.
- `do:*` → raw turbo-free executors (`astro build`, `wrangler deploy …`). Directly callable; no turbo, no rebuild when invoked as `pnpm run do:*`.

CF Workers Builds settings:

- **Build command**: `pnpm build` — turbo is fine here (no remote-cache token just means a cache miss; `astro build` runs fresh).
- **Deploy command**: `pnpm run do:versions:upload` (or `pnpm run do:deploy` for prod-direct) — turbo-free and build-free; it uploads the `dist/` the build command already produced. Never `pnpm run deploy`/`versions:upload` here — those re-enter turbo and its `do:build` dep re-runs the build inside CF's deploy sandbox.

Deploy targets `dist/server/wrangler.json` — the adapter's merged config emitted at build time; the root `wrangler.jsonc` alone would deploy the wrong (unbuilt) worker.

## Layout SSR — non-obvious gotcha

The body `<slot />` in `src/layouts/base-layout.astro` is **NOT** wrapped in any `client:*` React island. Wrapping it (any directive, including `client:load`) breaks first paint — Astro emits the slot's HTML, but React 19 sees a hydration mismatch (slot HTML ≠ React vnodes) and tears down the subtree. Stand-alone islands (e.g. `ConsentBanner`) live as **siblings** of the slot, not parents. Regression test: `src/__tests__/integration/base-layout-ssr.test.ts`.

`<slot name="head" />` is exposed in base-layout and forwarded through main-layout, so per-route `<Fragment slot="head">` (e.g. blog/index DNS prefetch) actually lands. Regression test: `src/__tests__/integration/layout-head-slot.test.ts`.

## Content & R2

- Bucket layout: `posts/<slug>.md`, `drafts/<slug>.md`, `assets/images/<slug>/<file>` in `articles-content`. Single bucket; `wrangler dev` reads it directly via `remote: true` on the R2 binding.
- `src/content.config.ts` wires `r2MarkdownLoader` directly. The loader rewrites legacy `../assets/images/<slug>/<file>` references to `/api/img/<slug>/<file>` so they resolve through the R2 streamer at runtime. Build fails loudly without R2 credentials — no glob fallback.
- Migration: `node scripts/migrate-articles-to-r2.mjs --source <path> --bucket <name> [--dry-run|--commit]`. Idempotent (md5 + ETag skip). Requires an R2 token with **read+write** scope.

## API surface

Two SSR endpoints only:

- `/api/img/[...path]` — streams images from the R2 `ARTICLES` bucket. `prerender = false`. Accessed via paths the R2 loader rewrites in markdown frontmatter and bodies.
- `/api/health` — minimal liveness probe. Returns `{ status: 'healthy', timestamp }` with `cache-control: no-store`. No CORS allow-all, no request-header echo.

`wrangler.jsonc → assets.run_worker_first: ["/api/*"]` ensures these hit the worker before the static-asset binding 404s them.

## Secrets / env

`.env` is the single source. `.envrc` (committed) hooks direnv so vars load on `cd` into the project. Build-time vars (Sentry, R2, Turbo) land in CI via Workers Build env. There are no runtime secrets — the worker reads only `vars` declared in `wrangler.jsonc`.

## Sentry on Cloudflare

Sentry MUST be the first integration in `astro.config.mjs`. Request handler auto-instrumentation is off for workerd compatibility.

`sentry.{client,server}.config.ts` gate `Sentry.init()` on environment detection — workerd rejects `addEventListener('load', _, true)` from `browserTracingIntegration`, so we skip init on workerd (`globalThis.WebSocketPair`) and outside browsers. Sentry stays live in real Node SSR and real browsers.

## React 19 + bundler


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raisedadead/portfolio](https://github.com/raisedadead/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
