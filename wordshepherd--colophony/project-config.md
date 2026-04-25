---
trigger: always_on
description: Open-source suite covering the full publication lifecycle: submission intake, publication pipeline, notifications, and cross-instance federation.
---

# Colophony — Infrastructure for Literary Magazines

Open-source suite covering the full publication lifecycle: submission intake, publication pipeline, notifications, and cross-instance federation.

**Status:** v2 rewrite in progress. v1 MVP (originally named Prospector) tagged as `v1.0.0-mvp`.
**Team:** David (primary dev), Senior Developer (PR reviews), CEO (priorities)
**Session log:** `docs/devlog/YYYY-MM.md` (monthly rotation) — append entries after each session
**Architecture:** [docs/architecture.md](docs/architecture.md)

### Suite Components

| Component    | Scope                                                                     | Package prefix          |
| ------------ | ------------------------------------------------------------------------- | ----------------------- |
| **Hopper**   | Submission management — forms, intake, review pipeline, decisions         | `@colophony/hopper-*`   |
| **Slate**    | Publication pipeline — copyedit, contracts, issue assembly, CMS           | `@colophony/slate-*`    |
| **Relay**    | Notifications & communications — email, webhooks, in-app messaging        | `@colophony/relay-*`    |
| **Register** | Identity & federation — cross-instance identity, sim-sub, piece transfers | `@colophony/register-*` |

API layer and plugin system use the `@colophony/` prefix directly.

---

## Key File Locations

Per-directory CLAUDE.md files contain domain-specific details:

- **`packages/db/CLAUDE.md`** — RLS rules (authoritative), schema files, migration workflow
- **`apps/api/CLAUDE.md`** — Hook registration, tRPC procedures, auth, webhooks
- **`apps/web/CLAUDE.md`** — tRPC client, providers, auth utilities, conventions

Full file index: [docs/file-index.md](docs/file-index.md) | Project structure: [docs/architecture.md](docs/architecture.md)

---

## Tech Stack

**Frontend:** Next.js 16, React 19, TypeScript (strict), Tailwind + shadcn/ui (New York), tRPC (internal), TanStack Query, tus-js-client, date-fns

**Backend:** Fastify 5, TypeScript (strict), Drizzle ORM, BullMQ 5, Zitadel (auth), Stripe, nodemailer

**API surfaces:** tRPC (built), oRPC REST + OpenAPI 3.1 (built). GraphQL (Pothos + Yoga) extracted to feature branch — re-merge when demand materializes

**Data:** PostgreSQL 16+ (RLS via Drizzle `pgPolicy`), Redis 7+, Garage (S3-compatible)

**Infra:** Docker Compose (self-hosted), Caddy (TLS + routing), Hetzner VPS

---

## Critical Patterns

### 1. Multi-Tenancy with RLS (CRITICAL)

**See `packages/db/CLAUDE.md`** for the authoritative RLS rules, `withRls()` usage, code examples, and full NEVER list.

Summary: RLS policies are in Drizzle schema via `pgPolicy`. Org context set via `SET LOCAL` inside transactions. Never query tenant data without setting context. Never make `app_user` a superuser.

### 2. Authentication (Zitadel OIDC)

**See `apps/api/CLAUDE.md`** for hook chain, token types, and auth details.

Summary: Zitadel handles all authentication. API validates tokens via Fastify `onRequest` hook. User lifecycle synced via Zitadel webhooks.

### 3. File Uploads (tusd)

Client → tusd sidecar (chunked, resumable) → pre-create hook validates → post-finish hook creates record → BullMQ → ClamAV scan → clean/quarantine. Uses tus-js-client on frontend, tusd sidecar in Docker Compose.

### 4. Payments (Stripe Checkout)

**See `apps/api/CLAUDE.md`** for PCI NEVER list and webhook idempotency patterns.

Summary: Stripe Checkout only (zero PCI scope). Two-step idempotency (INSERT event, check `processed` status). Never log card numbers or store card data.

### 5. Frontend

**See `apps/web/CLAUDE.md`** for tRPC client setup, providers, auth utilities, and conventions.

---

## Known Quirks & Gotchas

Domain-specific quirks are in per-directory CLAUDE.md files. Cross-cutting quirks below:

| Quirk                                                 | Details                                                                                                                                                                              |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Docker Compose env_file**                           | `env_file:` sets container env only. For YAML `${VAR}` substitution, use `--env-file .env` on CLI                                                                                    |
| **PostgreSQL init-db.sh**                             | Only runs on first DB creation. `docker compose down -v` to re-run                                                                                                                   |
| **PgBouncer: migrations must bypass**                 | `DATABASE_URL` (port 5432) for migrations. `DATABASE_APP_URL` (port 6432) through PgBouncer. Use `SET LOCAL` (not `SET`) — transaction pooling reuses connections                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wordshepherd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
