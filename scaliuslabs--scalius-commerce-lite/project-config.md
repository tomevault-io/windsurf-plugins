---
trigger: always_on
description: Turborepo monorepo: TanStack Start admin dashboard + Astro SSR storefront + standalone Hono API, all deployed as Cloudflare Workers. Admin and storefront communicate with the API worker through Cloudflare Service Bindings in production, with HTTP fallbacks for local development.
---

# Scalius Commerce

## Overview

Turborepo monorepo: TanStack Start admin dashboard + Astro SSR storefront + standalone Hono API, all deployed as Cloudflare Workers. Admin and storefront communicate with the API worker through Cloudflare Service Bindings in production, with HTTP fallbacks for local development.

## Monorepo Structure

```
apps/
  admin-v2/       # @scalius/admin-v2 — TanStack Start admin dashboard Worker
  api/            # @scalius/api — Hono API Worker, queues, cron, WidgetDesignAgent DO
  storefront/     # @scalius/storefront — Astro 7 SSR storefront Worker
packages/
  api-client/     # @scalius/api-client — generated SDK from OpenAPI spec
  core/           # @scalius/core — domain services, auth, providers, integrations, search
  database/       # @scalius/database — Drizzle schema, D1 client, migrations
  shared/         # @scalius/shared — shared utilities
  tsconfig/       # @scalius/tsconfig — shared TS config package
```

## Quick Commands

| Command | Purpose |
|---------|---------|
| `pnpm dev` | Start API :8787, admin :4323, storefront :4322 via `scripts/dev.sh` |
| `pnpm dev:all` | Alias for `pnpm dev` |
| `pnpm dev:api` | Start API :8787 via `scripts/dev.sh` with local migrations/readiness |
| `pnpm dev:admin` | Start admin :4323 + API :8787 |
| `pnpm dev:storefront` | Start storefront :4322 + API :8787 |
| `pnpm build` | Run `prebuild` (`scripts/copy-flags.mjs`) then Turbo build for workspaces with build scripts |
| `pnpm typecheck` | Run type checks through Turbo |
| `pnpm lint` | Run lint through Turbo for the seven code workspaces |
| `pnpm test` | Run all Vitest tests directly with `vitest run --passWithNoTests` |
| `pnpm test:watch` | Run Vitest in watch mode |
| `pnpm dev:setup` | Install deps, create local env files, apply local D1 migrations, create default local admin |
| `pnpm dev:reset` | Wipe local state, re-apply migrations, recreate default local admin |
| `pnpm dev:admin:create` | Create default local admin if none exists |
| `pnpm dev:admin:reset` | Reset local auth/admin credentials without wiping catalog/order data |
| `pnpm dev:admin:status` | Check whether a local admin exists |
| `pnpm dev:doctor` | Non-mutating local readiness check for env files, shared secrets, ports, services, and Wrangler state |
| `pnpm dev:doctor:api` | Require local API to be live |
| `pnpm dev:doctor:admin` | Require local API + admin to be live |
| `pnpm dev:doctor:storefront` | Require local API + storefront to be live |
| `pnpm dev:doctor:all` | Require API + admin + storefront to be live |
| `pnpm db:generate` | Generate Drizzle migrations via the API workspace |
| `pnpm db:migrate:local` | Apply D1 migrations to local Wrangler state |
| `pnpm db:migrate:remote` | Apply D1 migrations to remote D1 |
| `pnpm db:studio` | Drizzle Studio DB browser |
| `pnpm --filter @scalius/database check:migrations` | Verify migration SQL/journal/snapshot metadata and the manual snapshot-gap allowlist |
| `pnpm check:env` | Verify Wrangler binding/var names match Worker `Env` declarations |
| `pnpm check:dist-secrets` | Fail if app `dist/` outputs contain local env files such as `.dev.vars` or `.env*` |
| `pnpm generate:sdk` | Regenerate API client from the API OpenAPI spec |
| `pnpm run deploy` | Typecheck, build, migrate remote D1, deploy API + admin + storefront |
| `pnpm run deploy:api` | Typecheck, build API, migrate remote D1, deploy API |
| `pnpm run deploy:admin` | Typecheck, build admin, deploy admin |
| `pnpm run deploy:storefront` | Typecheck, build storefront, deploy storefront |
| `pnpm run deploy:api -- --dry-run` | Typecheck, build, and dist-check API without applying D1 migrations or deploying |

## Architecture

### Apps

- **Admin (`apps/admin-v2/`)**: TanStack Start + React 19 admin dashboard. Server functions live in typed domain slices under `src/lib/api-functions/`; query options live in domain modules under `src/lib/api-query-options/`; mutation hooks live in domain modules under `src/lib/api-mutations/`, with `api.mutations.ts` kept only as a compatibility re-export. The broad `api.queries.ts` file has been removed. Use fresh `rg` scans for volatile counts instead of trusting prose. Uses `env.API` service binding in production and Vite proxy/HTTP to `localhost:8787` in local dev. Also has direct D1/KV/R2 bindings for auth, RBAC, and storage initialization.
- **API (`apps/api/`)**: Standalone Hono `OpenAPIHono` app mounted at `/api/v1`. Exports a `WorkerEntrypoint` with `fetch`, `queue`, and scheduled maintenance for orphan inventory expiry, stale hosted-payment cleanup, old/empty abandoned-checkout pruning, and notification-outbox flushing. Owns public/admin API routes, webhook ingestion, OpenAPI spec, queue consumer, and `WidgetDesignAgent` Durable Object for widget AI generation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scaliuslabs/scalius-commerce-lite](https://github.com/scaliuslabs/scalius-commerce-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
