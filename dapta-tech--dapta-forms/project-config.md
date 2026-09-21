---
trigger: always_on
description: This file is loaded automatically by Claude Code for anyone working in this repo.
---

# Dapta Forms — guide for Claude Code (and any AI coding agent)

This file is loaded automatically by Claude Code for anyone working in this repo.
It is the operating summary: how to run, test, and change **Dapta Forms** without
breaking the invariants that keep the project self-hostable and open. Deeper
rationale (request flow, package boundaries, the ports/adapters seams) lives in
[`ARCHITECTURE.md`](ARCHITECTURE.md); contribution mechanics (DCO, PR gates) live
in [`CONTRIBUTING.md`](CONTRIBUTING.md). Read those two when this summary points
you at them.

> **Naming:** the product is **Dapta Forms**. Internal packages use the `@quill/*`
> scope — `quill` is the codename, nothing more. Prefer "Dapta Forms" in
> user-facing text and `@quill/*` only when naming a package.

## What this is

An open-source, self-hostable forms platform: multi-step forms with skip-logic,
lead scoring, outcome buckets, partial + complete submissions, a first-party
funnel-event stream, durable email notifications, and short shareable links. A
Turborepo + pnpm-workspaces monorepo of two apps and seven packages. **Postgres
is the source of truth** (CI and production); **SQLite is a zero-infra dev
accelerator** so a bare clone runs in seconds.

## Repo map

```
apps/
  web/   Next.js 16 App Router (RSC) — public form pages + admin dashboard + builder
  api/   NestJS — public form API, admin/host API, outbox worker
packages/
  types/          zod contracts shared by web + api (formConfig v1, submissions, events)
  engine/         pure form logic — skip-logic, validation, scoring, outcomes (NO I/O)
  db/             Drizzle schema (pg + sqlite) + migrations + seed + repositories
  notifications/  EmailProvider port + adapters (log-only/noop/smtp/http) + notifier
  destinations/   SubmissionDestination port + adapters (webhook/hubspot/log-only)
  shared/         i18n (en/es), handle utils, growth attribution, design tokens
  config/         zod env schema + shared tsconfig/prettier presets
```

Dependency direction is one-directional: **apps depend on packages, never the
reverse; the web app reaches the API only over HTTP** (it never imports `@quill/db`
or the engine for data at runtime). The engine is pure and shared by both apps, so
the client-side preview and the server-side verdict always agree.

## How to run (zero-infra SQLite path)

```bash
pnpm install     # Node >= 20, pnpm >= 10
pnpm dev         # builds packages, migrates + seeds a SQLite DB, starts web + api
```

`pnpm dev` runs `db:setup` (migrate + seed) **before** launching the apps, so the
schema always exists. Then open:

- **Web** → http://localhost:3000 — seeded demo form at
  `/acme/alex-rivera/lead-qualifier`
- **API** → http://localhost:4000/health → `{"status":"ok",…,"dialect":"sqlite"}`

No Docker, no Postgres, no accounts. The DB is a file at `.data/dev.db`; email is
`log-only` (submission notices print to the API log); dashboard auth is a local
stub (you are logged in as the seeded demo account). Reset demo data with
`pnpm db:reset`.

**Ports.** The defaults above are what `pnpm dev` binds. To relocate the **API**,
set `API_PORT` (the Nest app reads it at runtime) and point the web app at it with
`NEXT_PUBLIC_API_URL`. The **web** dev port is set by the `apps/web` dev script; to
run it elsewhere use `pnpm --filter @quill/web exec next dev -p <port>`. See
`.env.example` for every knob — all have safe defaults.

**Postgres parity mode** (matches CI + production):

```bash
pnpm dev:pg            # docker compose up postgres, migrate + seed, run both apps
PG_PORT=5433 pnpm dev:pg   # if 5432 is taken locally
```

The `.claude/skills/local-dev` skill wraps boot / seed / login / reset / parity as
an invokable recipe if you'd rather not remember the commands.

## How to test

Vitest across the board (no Jest). Run from the repo root:

```bash
pnpm test            # all packages + apps (SQLite)
pnpm typecheck
pnpm lint
pnpm build           # builds packages AND both apps (what CI builds)
```

Scope to one workspace, or one file:

```bash
pnpm --filter @quill/engine test
pnpm --filter @quill/engine exec vitest run src/form-logic.spec.ts
```

**Postgres parity test** (the submission-integrity path CI asserts on every PR):

```bash
docker compose up -d --wait db
DATABASE_URL=postgres://quill:quill@localhost:5432/quill pnpm db:migrate
DATABASE_URL=postgres://quill:quill@localhost:5432/quill pnpm db:seed
DATABASE_URL=postgres://quill:quill@localhost:5432/quill pnpm --filter @quill/db test
```

`packages/db/src/submission.spec.ts` runs against `DATABASE_URL` on **both**
dialects and asserts the `submission_form_session_uq` unique index rejects a
duplicate — that is the SQLite↔Postgres parity guarantee. If you touch the DB
layer, run this before you push.

## Architecture invariants (a change MUST respect these)

1. **Dual-dialect schema parity + additive-only migrations.** The data model lives
   in two files — `packages/db/src/schema.pg.ts` (source of truth) and
   `packages/db/src/schema.sqlite.ts` (portable subset). They mirror 1:1 on
   table/column names (Postgres `jsonb`/`bigint` ↔ SQLite `text` JSON/`integer`
   epoch-ms). Any schema change edits **both**, ships a numbered migration in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dapta-Tech/dapta-forms](https://github.com/Dapta-Tech/dapta-forms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
