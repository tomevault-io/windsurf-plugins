---
trigger: always_on
description: Anonymous love letters app with disappearing ink. T3 Turbo stack + Supabase (as the
---

# Agent Instructions

Anonymous love letters app with disappearing ink. T3 Turbo stack + Supabase (as the
Postgres host — see "Architecture decisions" before assuming anything else about it).

## Agent-driven development

`AGENTS.md` is the tool-agnostic guide: provisioning that actually works from a fresh
clone, how to verify a change end-to-end (`pnpm verify` plus a browser recipe against the
real UI), and which surfaces an agent can check at runtime at all. **Read it before
starting work**; it is meant to be run, not skimmed. This file stays the Claude-facing
reference for conventions and for the architecture decisions below — the two do not
duplicate each other.

## Stack

- **Monorepo**: pnpm + Turborepo
- **Web**: Next.js (App Router, Turbopack), Tailwind v4, tRPC
- **Mobile**: Expo / React Native (`apps/expo`) — the shipping native app
- **Legacy mobile**: Capacitor (`apps/mobile`) — superseded WebView shell, plus the
  `@capacitor/*` runtime inside `apps/web` (see "Architecture decisions"). Do not add
  features; do not delete either without reading "Architecture decisions" first.
- **DB**: Postgres (Supabase) + Drizzle ORM
- **Auth**: hand-rolled signed-cookie sessions (`packages/api/src/auth/session.ts`).
  **Not Supabase Auth.** Supabase is the Postgres host and local CLI only.
  Read "Architecture decisions" before changing anything here.
- **Notifications**: Knock
- **Hosting**: Vercel

## Structure

```
apps/
  web/         # Next.js web app
  expo/        # iOS/Android native app (Expo) — the one that ships
  mobile/      # Legacy Capacitor shell — superseded, pending removal. Its Android
               # applicationId still matches the live Play Store package, so it is
               # the only source that can rebuild the legacy Android artifact.
packages/
  api/         # tRPC routers + auth/session
  contracts/   # SHARED domain: zod schemas + pure rules used by BOTH web and expo.
               # Shared domain logic belongs HERE, not duplicated per-platform.
  db/          # Drizzle schema, Postgres client
  ui/          # shadcn-ui components (web only)
```

## Commands

```bash
pnpm dev              # All packages (turbo watch)
pnpm dev:web          # Web only
pnpm dev:expo         # Expo only
pnpm db:start         # Start local Supabase
pnpm db:stop          # Stop Supabase
pnpm db:reset         # Reset DB
pnpm db:push          # Push schema to local
pnpm db:push-remote   # Push schema to production
pnpm db:seed          # Perf fixture — large, NOT idempotent, no signable accounts
pnpm verify           # typecheck + lint + format + test — mirrors CI, run before commit
pnpm lint             # oxlint (NOT ESLint), warnings are errors
pnpm format           # oxfmt --check (use format:fix to write)
pnpm typecheck        # TypeScript
pnpm test             # node:test suites (turbo run test)
pnpm build            # Build all
```

## DB (packages/db)

```bash
pnpm -F db studio     # Drizzle Studio
pnpm -F db seed       # Run seed script
pnpm -F db apply-sql  # Re-apply sql/ only (push already does this)
```

**The schema is declared, not migrated. There are no migrations.** It has two halves,
and `pnpm db:push` applies both, in this order:

1. `src/drizzle-schema.ts` — tables, columns, indexes. `drizzle-kit push` syncs these.
2. `sql/*.sql` — functions, triggers, grants, **and views**. `src/apply-sql.ts` runs
   every file, in filename order, in one transaction. Every file is idempotent; it
   re-runs on every push. See `sql/README.md`.

`pnpm db:push` is therefore the whole deploy, and `pnpm db:reset` is
`supabase db reset && push && seed`. Nothing is replayed, so no migration history can
drift from the schema.

The LOCAL `push` runs `drizzle-kit push --force` so a headless run cannot hang on the TTY
confirmation prompt. `--force` accepts data-loss statements without asking, which is fine
against a disposable local database and is why `push:remote` deliberately does **not** carry
it — production stays interactive.

**`drizzle-kit push` does NOT diff a view's body.** This is the trap. It creates a view
that is missing and drops one deleted from the schema file, but when the name already
exists it emits _nothing_, however much the SELECT changed — exit 0, no warning.
Verified against a production-shaped database. So: **any view is declared `.existing()`
in `drizzle-schema.ts` (for column types only) and its DDL lives in `sql/090-views.sql`.**
A view written with `.as(...)` will silently rot in production while push reports
success. `main` shipped a `Feed` whose drizzle copy had already drifted from the
deployed view (it was missing `AND p."parentId" IS NULL`, which would have put every
comment in the feed); it never broke anything _only_ because push ignored it.

Corollary: **push cannot roll a view back either.** Reverting the schema file and
pushing will leave the new view live. A rollback needs explicit SQL.

The old `generate`/`migrate` scripts are gone, which retired the standing footgun that
generate emitted `DROP TABLE "auth"."users" CASCADE` into every migration (an artifact
of `schemaFilter: ["public"]` hiding Supabase's `auth` schema). `push` never had that
bug — verified, it emits no `auth` DDL.

## Testing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kyh/yours-sincerely](https://github.com/kyh/yours-sincerely) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
