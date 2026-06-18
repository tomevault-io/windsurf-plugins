---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project

Taskwondo — a self-hosted task and ticket management system. Monorepo with a Go REST API (`api/`), React frontend (`web/`), MCP server (`mcp/`), and Playwright E2E tests (`test/e2e/`).

## Commands

Run `make help` for the full list of Make targets (dev, test, build, migrate, release, etc.).
Requires `.env` — copy from `.env.template`.

Commands not covered by `make help`:

```bash
# Go tests — single package / single test
cd api && go test ./internal/handler/... -v -race
cd api && go test ./internal/service/... -v -run TestName

# Frontend — lint / typecheck only (no Make target)
cd web && npm run lint
cd web && npm run typecheck

# Migrations also run automatically on API startup.
# Use `./taskwondo --migrate-only` to run migrations and exit (useful for init containers / CI).
```

## Architecture

### Go API (`api/`)

Entry point: `api/cmd/server/main.go`. Internal packages follow `handler → service → repository` dependency direction (never reversed). Interfaces are defined by the consumer.

```
api/internal/
  config/       — Env-based configuration
  database/     — DB connection + migration runner
    migrations/ — Numbered SQL files (000001_*.up.sql / *.down.sql), append-only
  handler/      — HTTP handlers (chi router), DTOs, request/response parsing
  middleware/   — Auth (JWT + API key), CORS, logging, rate limit, etc.
  model/        — Domain structs + error sentinels (ErrNotFound, ErrForbidden, ErrConflict, ErrValidation, ErrInvalidTransition)
  repository/   — SQL queries implementing service interfaces
  service/      — Business logic, RBAC authorization
  storage/      — Storage interface + MinIO/S3 implementation (attachments)
```

### React Frontend (`web/src/`)

```
api/          — Axios client functions (one file per domain)
components/ui/— Reusable primitives (Button, Input, Modal, Badge, DataTable, etc.)
components/workitems/ — Domain components (BoardView, CommentList, WorkItemForm, etc.)
contexts/     — Auth, Theme, Language, Notification contexts
hooks/        — TanStack Query hooks (useWorkItems, useProjects, useWorkflows, etc.)
i18n/         — en.json (all UI strings), init config
pages/        — Page components
```

Path alias: `@/` → `src/`. Vite proxies `/api` to `:8080` in dev.

### Key Patterns

- **Routing**: chi router. URL identifiers are project keys (not UUIDs): `/projects/:projectKey/items/:itemNumber`
- **Work item numbers**: Per-project sequential integers, incremented atomically during insert
- **IDs**: UUIDv7 for time-ordered entities (work items, events), UUIDv4 elsewhere
- **Auth**: JWT + API key (`twk_<hex>`) middleware. Passwords bcrypt-hashed, API keys SHA-256 hashed.
- **Pagination**: Cursor-based (last item ID), not page numbers
- **Soft deletes**: All queries filter `WHERE deleted_at IS NULL`
- **Workflow statuses**: Categories (todo, in_progress, done, cancelled) drive resolved_at and board column logic

## Conventions

### Go
- **Logging**: zerolog only. Use `log.Ctx(ctx)` for contextual logging.
- **Context**: `context.Context` as first param everywhere (`_ context.Context` if unused)
- **Interfaces**: Define in the consumer package, not the provider. `service` defines repo interfaces; `repository` implements them.
- **Errors**: Wrap with context: `fmt.Errorf("creating work item: %w", err)`. For user-facing validation errors that need localization, use `model.NewKeyedError(sentinel, "error_key", "english message", params)` — the handler layer automatically extracts the key via `writeErrorFromService`.
- **Error keys**: Stable, snake_case identifiers (e.g. `namespace_slug_reserved`, `project_key_in_use`). Never rename once released. Add the corresponding `errors.<key>` i18n entry to all language files.
- **No global state.** Dependency injection via constructors. No `init()` except in `main`.
- **All times UTC** in the database. Convert to user timezone only in the frontend.
- **Commit messages**: Prefix with `[DISPLAY_ID]` (e.g. `[TF-141]`, `[PROJ-23]`) when a work item display ID is provided. The display ID format is `<PROJECT_KEY>-<NUMBER>`. No Co-Authored-By.

### React/TypeScript
- **i18n**: All UI strings in `web/src/i18n/en.json`. Use `const { t } = useTranslation()` in every component. `<Trans>` for JSX with embedded HTML. Module-level arrays with display strings must be inside component body. Interpolation: `{{var}}`. Pluralization: `_one`/`_other` suffixes. Any key added to `en.json` must also be added to all other language files.
- **Adding a new locale**: four places must stay in sync — (1) create `web/src/i18n/<code>.json` with a full translation of `en.json`, (2) import it in `web/src/i18n/index.ts` and add it to the `resources` map, (3) add the code to the `Language` type union in `web/src/contexts/LanguageContext.tsx`, (4) add an entry to the `SUPPORTED_LANGUAGES` array in the same file. The i18n Vitest suite will fail if keys drift between locales.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcoshack/taskwondo](https://github.com/marcoshack/taskwondo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
