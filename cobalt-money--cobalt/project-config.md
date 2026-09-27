---
trigger: always_on
description: Cobalt is a full-stack web application built as a Turborepo monorepo with Bun as the package manager.
---

# Cobalt Web

Cobalt is a full-stack web application built as a Turborepo monorepo with Bun as the package manager.

## Tech Stack

- **Runtime:** Bun
- **Monorepo:** Turborepo
- **Frontend:** Vite + React 19 + TanStack Router/Query/Start + Tailwind CSS v4
- **Backend:** Hono (on Bun)
- **Database:** PostgreSQL + Drizzle ORM
- **Real-time sync:** Rocicorp Zero
- **Auth:** Better Auth
- **UI components:** Shadcn (via @cobalt-web/ui)
- **Docs:** Fumadocs + MDX
- **Linting/Formatting:** Ultracite (Oxlint + Oxfmt)
- **Language:** TypeScript (ESM throughout)

## Monorepo Structure

```
apps/
  web/        — Main frontend app (Vite + TanStack Start)
  server/     — API server (Hono on Bun)
  fumadocs/   — Documentation site (Fumadocs + TanStack Start)
packages/
  auth/       — Better Auth configuration
  config/     — Shared base tsconfig
  db/         — Drizzle ORM schema, migrations, and client
  env/        — Zod-validated environment variables (server + web)
docs/
  local-sync/ — Local Postgres via Docker, env wiring, troubleshooting
  ui/         — Shared React components (Shadcn) and styles
  zero/       — Rocicorp Zero schema, queries, and mutators
.sandbox/
  ztunes/     — Zero sample app (reference patterns; optional, not in the default Turborepo pipeline)
```

## Workflow

After finishing a feature or making changes, always run:

```sh
bun check
```

This runs Ultracite (lint/format) and then `turbo check-types` (TypeScript) across the repo. To auto-fix issues:

```sh
bun fix
```

## Key Commands

| Command              | Description                                                      |
| -------------------- | ---------------------------------------------------------------- |
| `bun dev`            | Start all apps in dev mode                                       |
| `bun dev:web`        | Start only the web app                                           |
| `bun dev:server`     | Start only the server                                            |
| `bun build`          | Build all apps                                                   |
| `bun check`          | Ultracite (lint/format) then `turbo check-types` (TypeScript)    |
| `bun fix`            | Auto-fix lint and format issues                                  |
| `bun db:push`        | Push schema changes to database                                  |
| `bun db:generate`    | Generate Drizzle migrations                                      |
| `bun db:migrate`     | Run database migrations                                          |
| `bun db:studio`      | Open Drizzle Studio                                              |
| `bun zero:generate`  | Regenerate `packages/zero` schema from Drizzle (`drizzle-zero`)  |
| `bun db:local:up`    | Start local Postgres (see `docs/local-sync/`)                    |
| `bun db:local:down`  | Stop local Postgres                                              |
| `bun db:local:reset` | Recreate local Postgres (wipes volume)                           |
| `bun db:local:init`  | Postgres bootstrap before migrations (`packages/db/planetscale`) |
| `bun sync-env`       | Copy `.env*` from main Git worktree into this worktree           |
| `bun sync-sandbox`   | Copy `.sandbox/` from main worktree (gitignored reference apps)  |

## Skills

The `.agents/skills/` directory contains detailed guides for specific tools. Read the relevant skill before building features with that tool.

To discover and update skills shipped by npm dependencies, run:

```sh
npx @tanstack/intent install
```

| Skill                  | Path                                             | Use when                                                            |
| ---------------------- | ------------------------------------------------ | ------------------------------------------------------------------- |
| Hono                   | `.agents/skills/hono/SKILL.md`                   | Building or modifying server routes, middleware, or API endpoints   |
| Drizzle ORM            | `.agents/skills/drizzle-orm/SKILL.md`            | Working with database schema, queries, relations, or migrations     |
| PostgreSQL             | `.agents/skills/postgres/SKILL.md`               | Query optimization, connection troubleshooting, performance         |
| Tailwind Design System | `.agents/skills/tailwind-design-system/SKILL.md` | Design tokens, component variants, theming, responsive patterns     |
| Rocicorp Zero          | `.agents/skills/rocicorp-zero/SKILL.md`          | Zero, ZQL, zero-cache, sync queries, mutators, providers, debugging |

## Package Documentation

When building a new feature or need API reference, read the package source code directly rather than guessing. Look at `.d.ts` type definitions for API surface and exported functions. Read the actual source for implementation details and usage patterns.

| Package         | Source path                                                                                              |
| --------------- | -------------------------------------------------------------------------------------------------------- |
| Hono            | `node_modules/hono/dist/types/`                                                                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cobalt-Money/Cobalt](https://github.com/Cobalt-Money/Cobalt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
