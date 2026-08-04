---
trigger: always_on
description: Guidance for AI agents (and humans) working in this repo. This is the single source
---

# AGENTS.md

Guidance for AI agents (and humans) working in this repo. This is the single source
of truth — `CLAUDE.md` is a symlink to this file, so Claude Code loads it automatically
and other agents (Cursor, Codex, etc.) read `AGENTS.md` directly. **Keep it accurate —
see [Maintaining this file](#maintaining-this-file).**

## What this is

**fana** — a self-hostable disposable / temporary email service.
Random address in, mail arrives live, auto-expires. OSS, MIT.

> ⚠️ Inboxes are **public by design** — anyone who knows an address can read it.

## Stack

- **pnpm** workspaces + **Turborepo**, TypeScript strict, Node ≥ 22
- **Postgres** via **Drizzle ORM** (single dialect — MySQL support was considered and declined)
- **Redis** for realtime pub/sub + rate-limit
- Inbound **SMTP** (`smtp-server` + `mailparser`), API on **Hono** + `ws`, web on **Next.js 15** + Tailwind

## Layout

```
apps/
  smtp/   inbound SMTP → parse → store → publish realtime event
  api/    REST + WebSocket, per-IP rate-limit, admin (accounts + sessions), TTL purge job
  web/    Next.js realtime inbox UI + /admin dashboard (light default)
packages/
  core/   shared types, address generation, config + branding helpers
  db/     Drizzle schema + client (Postgres)
infra/    Caddyfile (reverse proxy, auto-HTTPS)
```

Inbound mail is authenticated at ingestion with **mailauth** (SPF/DKIM/DMARC) using
the SMTP session (IP/HELO/MAIL FROM). Results + a derived `verdict`
(`verified`/`unverified`/`suspicious`, see `packages/core/src/auth.ts`) are stored on
the message and shown as a badge in the UI. Never trust unauthenticated senders.

## Data flow

```
MX :25 → apps/smtp → simpleParser → Postgres (messages + attachments, bytea)
                                  → Redis PUBLISH event
apps/api  ← Redis SUBSCRIBE → fan out to WebSocket clients (/ws?mailbox=addr)
apps/web  ← REST (list/read) + WS (live new-mail) →
```

## Commands

```bash
pnpm install
docker compose up postgres redis -d   # datastores for local dev
cp .env.example .env                   # then set MAIL_DOMAINS
pnpm db:migrate                        # apply migrations (canonical)
pnpm dev                               # smtp + api + web in watch mode
pnpm typecheck && pnpm lint && pnpm test   # all must pass before a PR
docker compose up --build              # full stack
```

## Conventions (important)

- **Config comes from env, never hardcoded in tracked files.** Tracked files ship
  *generic* defaults (`example.com`, `localhost`); real values live in `.env` (gitignored).
  This is why `MAIL_DOMAINS`, `DB_*`, `SITE_ADDRESS` and `SITE_NAME` exist.
  Don't reintroduce a hardcoded domain or `DATABASE_URL`. Branding is the one exception
  to "generic defaults": the built-in theme in `packages/core/src/brand.ts` is the fallback
  every instance starts from, and `REPO_URL` there is a deliberate hardcoded attribution
  link — it is intentionally not themeable, so don't move it into env or the overrides.
- **DB config is discrete parts** (`DB_HOST/PORT/NAME/USER/PASSWORD/SSL`) built into a
  client in `packages/db` — not a connection URL.
- **Migrations are the source of truth** (`packages/db/drizzle/*.sql`). Edit `schema.ts`,
  run `pnpm db:generate` to create a migration, and **commit the SQL**. `pnpm db:migrate`
  applies pending migrations; the API also auto-migrates on boot (`MIGRATE_ON_START`).
  `db:push` exists only for throwaway local iteration — never rely on it for prod.
- **Table shape**: internal `id` (bigserial) PK + `created_at`/`updated_at` on every
  table. Rows exposed by the public API (messages, attachments) also carry an
  unguessable `public_id` (uuid) — the API returns/accepts `public_id`, NEVER the
  internal `id`, so nobody can enumerate rows by counting. Data is **hard-deleted**
  (ephemeral service) — no soft-delete/`deleted_at`.
- **Attachment blobs go through `@fana/storage`** (`getStorage()`), driver chosen by
  `STORAGE_DRIVER`: `db` (inline Postgres bytea, default) or `s3` (S3/R2/MinIO). An
  attachment row has exactly one of `content` (db) or `storageKey` (s3). All message
  deletion goes through `deleteMessagesWhere` so s3 objects are cleaned up too.
- **Served domains are dynamic**: built-in (`MAIL_DOMAINS`) ∪ verified community
  domains (`domains` table). SMTP + API each cache the set (`domains.ts`, refresh ~30s)
  — never query per-request. Community domains self-register via `POST /api/domains`;
  ownership is proven by the domain's MX pointing at `PUBLIC_MX_HOST` (`DOMAIN_VERIFY=off`
  auto-verifies for local dev). Reads stay public.
- **All DB access goes through `packages/db`** (`getDb()` + Drizzle). Queries currently
  live inline in routes/smtp; if that grows, extract a repository layer.
- **Addresses**: two paths.
  - *Random* (`packages/core/src/address.ts`, 3 styles, each ≥1e9 combos): minted +
    **reserved** server-side (`reservations` table), returns a token the client renews via
    `POST /mailbox/claim`. Reservation only stops the generator handing out duplicates.
  - *Explicit* (typed, or from a URL like `/alias@domain`): **public, used directly by the
    client — no reservation, no token, no "taken" error**. This is the generator.email model.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JastinXyz/fana](https://github.com/JastinXyz/fana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
