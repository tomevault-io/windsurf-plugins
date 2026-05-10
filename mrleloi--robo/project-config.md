---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workspace Layout

`C:\htdocs\robo` is a **multi-repo workspace**, not a monorepo. There is no root `.git` — each service folder is its own independent git repository with its own `package.json`, dependencies, CI, and Changesets versioning.

| Folder | Type | Port | Stack | Role |
|---|---|---|---|---|
| `ms-robo-admin/` | NestJS backend | 3002 | NestJS 11 + CQRS + TypeORM + PostgreSQL 17 | Admin API. **Sole schema owner.** |
| `ms-robo-web/` | NestJS backend | 3001 | Same as above | Client API. **Schema consumer** — never creates migrations. |
| `mfe-robo-admin/` | React frontend | 5173 | React 19 + Vite 7 + Tailwind 4 + HeroUI + Zustand + TanStack Query | Admin dashboard. Talks to `ms-robo-admin`. |
| `mfe-robo-web/` | React frontend | 5174 | Same as above | Client portal. Talks to `ms-robo-web`. |
| `ms-robo-dataloader/` | Placeholder | — | — | Empty repo (only `.git`, `.gitignore`, README). Do not put work here without confirming with the user. |
| `documents/` | Spec docs | — | — | Source-of-truth handover specs (`01-project-overview.md` … `07-business-logic-qa.md`). Read these before touching unfamiliar areas. |
| `tasks/feat/` | Feature work | — | — | Each numbered subfolder (e.g. `01_portfolio_export`, `02_datalake_modify`) contains spec / design / Q&A / `client-reply.md` files for one feature. See "Feature task convention" below. |
| `backup/` | Snapshots | — | — | Schema dumps, view exports. Read-only reference. |
| `infra/` | Infra config | — | — | `docker-compose.infra.yaml` (currently empty). |

**Implication**: When working in any service, `cd` into that folder before running commands. `package.json` scripts only exist inside service folders, never at the root.

## Common commands

All backend services use **Bun** (`>=1.0.0`); both frontends use **Node 22**. Run from the relevant service directory.

### Backend (`ms-robo-admin`, `ms-robo-web`)

```bash
bun install                      # install deps
bun run start:dev                # dev with watch (port 3002 / 3001)
bun run build                    # nest build → dist/
bun run start:prod               # run dist/main
bun run lint                     # eslint
bun run format                   # prettier
bun run test                     # jest (uses @swc/jest in ms-robo-admin, ts-jest in ms-robo-web)
bun run test:watch
bun run test:cov                 # coverage → ./coverage
bun run test:e2e                 # spins up docker-compose.e2e.yaml first
bun run test -- path/to/file.spec.ts            # run a single test file
bun run test -- -t "should do X"                # run tests matching name
```

### Backend DB (only in `ms-robo-admin`, the schema owner)

```bash
bun run db:generate              # generate a new migration from entity diff
bun run db:create                # create empty migration
bun run db:migrate               # run pending migrations
bun run db:revert                # revert last migration
bun run schema:drop              # drop all tables (destructive)
bun run seed:run                 # run seeders
bun run sync:sus8 -- --limit 10  # SUS8 datalake sync (one-shot CLI)
bun run db:sync                  # push entity files into ms-robo-web (see "Schema ownership")
bun run db:sync:check            # diff entities between admin and web (read-only)
```

### Frontend (`mfe-robo-admin`, `mfe-robo-web`)

```bash
bun install                      # bun works fine even though node is the runtime
bun run dev                      # vite dev (port 5173 / 5174)
bun run build                    # vite build (mfe-robo-web also runs tsc first)
bun run typecheck                # tsc --noEmit
bun run lint                     # eslint .
bun run lint:fix
bun run format                   # prettier
bun run knip                     # detect unused exports/files
bun run test                     # vitest run
bun run test:watch
bun run test:ui                  # vitest UI
bun run test path/to/file.test.tsx              # run a single test file
bun run test -- -t "label"                      # run tests matching name
```

## Schema ownership (critical rule)

`ms-robo-admin` is the **sole owner** of the database schema. All `@Entity` definitions, migrations, and seeds live there. `ms-robo-web` is a **consumer** — its `entities/` directory is a synced copy that should never be hand-edited and never has its own migrations.

To propagate a schema change:

1. Add/edit the entity in `ms-robo-admin/src/infrastructure/persistence/entities/...`.
2. `cd ms-robo-admin && bun run db:generate` to create the migration.
3. `bun run db:migrate` against the local DB.
4. `bun run db:sync` to push the updated entity files into `ms-robo-web/`.
5. Verify with `bun run db:sync:check`.

If `ms-robo-web` shows entity drift, run `db:sync:check` first — never patch `ms-robo-web` entities directly.

## Backend architecture (both `ms-robo-*` services)

Clean Architecture + CQRS, with the same layout in both backends:

```
src/
├── core/
│   ├── domain/{context}/         # Entities, value objects, domain errors
│   └── application/{context}/    # Commands, queries, handlers, ports (interfaces)
├── infrastructure/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrleloi/robo](https://github.com/mrleloi/robo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
