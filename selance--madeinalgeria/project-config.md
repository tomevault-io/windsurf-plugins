---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Before any frontend/UI work, read `.claude/rules/design-system.md`. Before finishing a task, follow `.claude/rules/workflow.md`.

## Commands

```sh
pnpm install
pnpm typecheck          # all workspaces (parallel)
pnpm lint               # oxlint (oxc.rs), whole repo — config in .oxlintrc.json
pnpm test               # all workspaces (serial — workers tests can't run concurrently)
pnpm --filter @mia/api test                          # one workspace
pnpm --filter @mia/module-billing exec vitest run src/foo.test.ts   # one file
pnpm dev:api            # public API, local miniflare :8787 — never --remote
pnpm dev:admin-api      # admin API :8788
pnpm --filter @mia/app dev        # user dashboard SPA (vite :5173)
pnpm --filter @mia/admin-web dev  # admin dashboard SPA (vite)
pnpm --filter @mia/web dev        # public site (astro :4321)
```

DB (from `apps/api`): `pnpm db:migrate:local|staging|production`, `pnpm seed:local|staging`. Seeding is CLI-only (`tools/seed`) — never add HTTP seeding endpoints. New schema → edit `packages/db-core/src/schema/*`, then `pnpm --filter @mia/db-core db:generate`.

Deploys: every app has `deploy:staging` / `deploy:production` (wrangler). `pnpm deploy:all` at the root does staging then production for every workspace. CI (`.github/workflows`) deploys everything to staging on push to `main`; production workflow is manual dispatch.

## Architecture

pnpm monorepo, modular monolith on Cloudflare Workers. Everything is TypeScript, Hono on the backend, React 19 on the frontend, Drizzle on D1.

**Apps** (`apps/*`, each a Worker):
- `api` — public API. Hono + all modules, queue consumer (`email.send`, `campaign.dispatch`), better-auth mounted on `/v1/auth/*`.
- `admin-api` — admin API. Same modules' admin routers, global `requireAdmin` before everything.
- `web` — public/SEO site (Astro static-first on a Worker). Middleware does not run for prerendered pages.
- `app` — user dashboard SPA. React Router (data mode), TanStack Query/Table, react-hook-form + zod.
- `admin-web` — admin dashboard SPA. Same stack as `app`.

**Packages** (`packages/*`):
- `core` — `AppError`/response envelope, D1 statement-limit helpers (`chunkedIn`, `batchInsert`), slugify, search normalization.
- `contracts` — zod schemas + service interfaces shared by API and clients. **Modules never import other modules — they talk through `@mia/contracts` only** (oxlint-enforced via `no-restricted-imports` in `.oxlintrc.json`).
- `db-core` — Drizzle schema (auth/profiles/billing/email/reference) + migrations. One D1 database.
- `auth` — better-auth factory (`createAuth`): Drizzle adapter, KV-backed session storage, email flows, `bearer()` + `admin()` plugins; `requireAdmin` Hono middleware.
- `email` — Resend-via-fetch sender (+ `consoleOutbox` dev sender), workerd-safe React Email rendering, KV rate limiter, transactional + newsletter templates.
- `api-client` — typed ky client shared by all frontends: envelope unwrap, `ApiError`, better-auth client (`createAppAuthClient`), `queryKeys` factory, `ApiClientProvider`.
- `ui` — the design system (see `.claude/rules/design-system.md`). Root export is utilities only; components are imported per file (`@mia/ui/components/button`).
- `modules/*` — billing, identity, notifications, reference.

**Module anatomy** (every `packages/modules/*`): `repo.ts` (the ONLY file that touches the db), `service.ts` (business logic), `router.ts` + `admin-router.ts` (thin Hono factories taking `getService`/`getUser` accessors), `index.ts` (public surface). The composition root is `apps/*/src/container.ts` — hand-written memoized factories, one instance per isolate.

**Adding a module** (there is deliberately no demo module; the kept ones are the reference):
1. Copy the anatomy of `packages/modules/billing` (simplest) or `notifications` (queue + email + signed tokens).
2. Schema → `packages/db-core/src/schema/<name>.ts`, export from `schema/index.ts`, `pnpm --filter @mia/db-core db:generate`.
3. Zod contracts → `packages/contracts/src/<name>.ts`.
4. Wire factories in `apps/api/src/container.ts` (and admin-api's if it has admin routes).
5. Mount in `apps/api/src/app.ts` — literal routes before param routes; per-route `requireUser` on protected subpaths.
6. Workerd integration test in `apps/api/test/<name>.test.ts`, modeled on `newsletter.test.ts` (`app.request(path, init, env)` against the real app + real migrations).

**Data rules** (non-negotiable):
- Any app-level `WHERE x IN (…)` goes through `chunkedIn`; bulk inserts through `batchInsert` (D1: 100 params / 100 KB SQL).
- API list endpoints use keyset pagination (`cursor` = last-seen key, fetch `limit+1` to compute `nextCursor`) and ship with their index + an `EXPLAIN QUERY PLAN` check. Offset `?page=` mode with totals only where a numbered UI needs it.
- Secrets only via `wrangler secret put --env <env>` / `.dev.vars` — never in `wrangler.jsonc` `vars`.

**Validation & envelope** (every endpoint):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [selance/madeinalgeria](https://github.com/selance/madeinalgeria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
