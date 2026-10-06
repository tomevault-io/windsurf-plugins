---
trigger: always_on
description: Codebase knowledge for any AI agent or reviewer. Agent workflow rules for Copilot live in `.github/copilot-instructions.md`.
---

# Streamarr Development Guide

Codebase knowledge for any AI agent or reviewer. Agent workflow rules for Copilot live in `.github/copilot-instructions.md`.

## Architecture

Hybrid **Next.js 16 + Express** app wrapping Plex/\*Arr services. Single process runs both servers.

| Layer       | Stack                                       | Location          |
| ----------- | ------------------------------------------- | ----------------- |
| Frontend    | Next.js App Router, React 19, SWR, Tailwind | `src/`            |
| Backend API | Express, TypeORM, SQLite                    | `server/`         |
| Real-time   | Socket.IO                                   | `server/index.ts` |

Express serves API routes (`/api/v1/*`) and delegates everything else to Next.js. See `server/index.ts`.

**Path aliases** (enforced by ESLint): use `@server/*` or `@app/*` for any import that crosses a top-level directory boundary (e.g., `server/routes/` importing from `server/lib/` must use `@server/lib/…`). Use relative `./` imports only when importing from within the same directory (e.g., `./utils` inside `server/routes/`).

- `@server/*` → `server/*`
- `@app/*` → `src/*`

## Build & Test

| Task               | Command                                                              |
| ------------------ | -------------------------------------------------------------------- |
| Dev (Node only)    | `pnpm dev`                                                           |
| Build all          | `pnpm build`                                                         |
| Pre-commit check   | `pnpm check` (format + lint + css-lint + typecheck)                  |
| Type check         | `pnpm typecheck` / `pnpm typecheck:server` / `pnpm typecheck:client` |
| Lint               | `pnpm lint` / `pnpm css-lint`                                        |
| Format             | `pnpm format`                                                        |
| E2E tests          | `pnpm cypress:build && pnpm cypress:open`                            |
| Generate migration | `pnpm migration:generate server/migration/Name`                      |
| Run migrations     | `pnpm migration:run`                                                 |
| Extract i18n       | `pnpm i18n:extract` (runs both `:client` and `:server` variants)     |
| API docs (dev)     | `http://localhost:3000/api-docs` (Swagger UI)                        |

Dev uses `nodemon` watching `server/**/*.ts` and `streamarr-api.yml`.

**Git hooks (Husky)**: `pnpm install` sets up Husky automatically via `bin/prepare.mjs`. Pre-commit runs `lint-staged` (format + lint on staged files). `commit-msg` enforces Conventional Commits via `commitlint`. Bypass with `HUSKY_BYPASS=1` if needed (e.g. CI sets `CI=true` to skip).

## Conventions

- Backend route conventions, authentication, OpenAPI, persistence, error handling, and serialization are in `.github/instructions/backend-api.instructions.md`.
- Frontend data fetching, state, i18n, styling, and mutation conventions are in `.github/instructions/frontend-components.instructions.md`.
- Workflow-specific release rules are in `.github/instructions/release-workflows.instructions.md`.

### Database Migrations

- **Dev**: `synchronize: true` — entity changes apply automatically in development. However, always run `pnpm migration:generate` before a production deploy whenever entity files change; synchronize does not create migration files and prod startup will fail if schema drifts without them.
- **Prod**: Migrations run automatically on startup. Generate from entity changes with `pnpm migration:generate`.

### Settings Migrations

A separate, lightweight migration system for JSON settings (not TypeORM). Files live in `server/lib/migrations/` and are run by `server/lib/migrator.ts` at startup before the settings singleton initialises. Each migration is a default-exported function `(settings: AllSettings) => AllSettings`. A backup (`settings.old.json`) is written before running. Use this when renaming or restructuring settings keys across versions.

## Key Files

| Purpose                     | File                                                                                                             |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| API spec (source of truth)  | `streamarr-api.yml`                                                                                              |
| Server entry point          | `server/index.ts`                                                                                                |
| DB config & `getRepository` | `server/datasource.ts`                                                                                           |
| Settings schema & defaults  | `server/lib/settings.ts`                                                                                         |
| Route registration          | `server/routes/index.ts`                                                                                         |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nickelsh1ts/streamarr](https://github.com/nickelsh1ts/streamarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
