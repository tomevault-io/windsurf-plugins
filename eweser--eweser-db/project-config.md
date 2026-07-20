---
trigger: always_on
description: > Required reading before work in this repository.
---

# EweserDB - AI Agent Instructions

> Required reading before work in this repository.

## Scope

These instructions apply to this repository and its subdirectories. They are for EweserDB only: the local-first, user-owned database project in this repo.

Do not mix instructions for unrelated projects into this file. If another project, such as an "outlandish web project", lives in a different repository, give it its own `AGENTS.md`. If it becomes part of this monorepo, add a narrower `AGENTS.md` in that project directory.

## Project Overview

EweserDB is a local-first, user-owned database SDK built on Yjs CRDTs. Users own their data. Apps interoperate over shared schemas and room-scoped access grants.

- Read `README.md` for product philosophy and API overview.
- Read `ARCHITECTURE.md` for current system design.
- Read `LOCAL_DEVELOPMENT.md` for setup and local service details.
- Treat `docs/ai/` and `docs/ai/adr/` as historical background unless a file explicitly says it is current guidance.
- Product/user configuration that must interoperate across apps should live in
  EweserDB rooms or shared schemas. PostgreSQL in `packages/auth-server-hono`
  should be reserved for auth, grants, sessions, OAuth, operational tokens, and
  security/audit metadata needed by the auth service.
- The product is not live with real users yet. When a plan explicitly touches
  pre-live product schemas or UX, prefer the clean long-term model over
  compatibility with unused prototype data. Still document intentional breaking
  changes, add migrations where PostgreSQL is involved, and keep published
  package changesets accurate.

## Current Architecture

The Next.js/Supabase migration is complete. The current stack is:

| Layer          | Current stack                                                       |
| -------------- | ------------------------------------------------------------------- |
| Core SDK       | TypeScript, Yjs, `y-indexeddb`, `@hocuspocus/provider`              |
| Shared package | TypeScript types, schemas, and helpers with no runtime dependencies |
| Auth API       | Hono, better-auth, Drizzle ORM, PostgreSQL                          |
| Auth UI        | React SPA built with Vite                                           |
| Sync server    | Hocuspocus relay with persistence                                   |
| Aggregator     | Server-side public indexing and search                              |
| Apps           | React 18-19, Vite, Tailwind CSS, Radix UI                           |
| Editor         | TipTap in `packages/ewe-note`                                       |
| Testing        | Vitest and Cypress                                                  |
| Build          | npm workspaces, Vite, `tsc`                                         |

Prefer changes that fit this architecture. Do not reintroduce Next.js or Supabase patterns unless the user explicitly asks and the tradeoff is documented.

## Hard Stops

- Never commit secrets, API keys, cookies, tokens, provider dashboard screenshots, or `.env` files.
- Never push directly to `main`. Use a branch and PR.
- Never delete database migrations. Only add new migrations.
- Never modify a published package API without a changeset.

## Monorepo Rules

- This is an npm workspaces monorepo. Check the root `package.json` before adding scripts or dependencies.
- Changes in `packages/shared` affect all consumers.
- Keep dependency versions unified across workspaces when practical.
- Use the repo's existing patterns before introducing new abstractions.
- Keep edits scoped. Avoid unrelated refactors.
- Use TypeScript strictly. Avoid `any`; if unavoidable, explain why in code or the final summary.
- For frontend work, preserve offline-first behavior and auth-grant flows.
- For backend work, validate inputs and keep auth boundaries explicit.

## Published Packages and Changesets

Run `npm run changeset` for behavior or API changes to published packages, including:

- `@eweser/db`
- `@eweser/shared`
- `@eweser/examples-components`

Docs-only, tests-only, internal app-only, and unpublished package changes usually do not need a changeset.

## Package Relationships

```text
@eweser/shared
    -> @eweser/db
        -> @eweser/examples-components
            -> examples / packages/ewe-note

packages/auth-server-hono is the Hono + better-auth auth API.
packages/app is the auth/account/access-grant React SPA.
packages/sync-server is the Hocuspocus sync relay.
packages/aggregator indexes public room data.
```

## Key Locations

| Location                            | Purpose                                             |
| ----------------------------------- | --------------------------------------------------- |
| `packages/shared/src/`              | Shared types, schemas, and helpers                  |
| `packages/db/src/`                  | Core SDK: rooms, documents, sync, local persistence |
| `packages/auth-server-hono/src/`    | Auth API                                            |
| `packages/app/src/`                 | Auth/account/access-grant SPA                       |
| `packages/sync-server/src/`         | Hocuspocus sync relay                               |
| `packages/aggregator/src/`          | Public indexing/search                              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eweser/eweser-db](https://github.com/eweser/eweser-db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
