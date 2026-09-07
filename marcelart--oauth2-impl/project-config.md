---
trigger: always_on
description: Guidance for AI agents working in this repo.
---

# AGENTS.md

Guidance for AI agents working in this repo.

## Hard rules

- **Do NOT run `serve`, `migrate`, `migrate-force`, `make dev`, `make go`, `make tw`, or any command that starts the server or touches the database.** These are for the human to run. Build/typecheck/compile edits only.
- **Do not cross the backend/frontend boundary in a single task unless the user explicitly asks for it.** Backend is `internal/v1`, `internal/app`, `internal/entities`, etc.; frontend is `internal/web/`. If a change genuinely requires both and the user didn't say so, stop and tell the user instead of doing both yourself.
- **Never fabricate library/package APIs.** Before writing code that uses any library (Gin, GORM, HTMX, Alpine.js, dig, cobra, swaggo, etc.), look up the real API via the Context7 MCP (`resolve-library-id` → `query-docs`). Do not rely on memory for signatures, generics syntax, or HTMX attributes.
- **Retain the existing project structure and layering.** Do not introduce new top-level packages or reorganize without being asked.
- **Authenticated in-app navigation must use HTMX partial swaps.** Clicking links, tabs, pagination, filters, form actions, or other controls inside the authenticated web shell must update only the relevant content region and must not reload or replace the whole document. Preserve the browser URL/history with `hx-push-url` when the destination represents a navigable page. Full-document responses remain required for direct URL loads and browser reloads; full navigation is also allowed when protocol or browser behavior requires it, such as login/logout boundaries, OAuth redirects to an OAuth Client, file downloads, or an explicit user request.

## Domain language

`CONTEXT.md` (repo root) defines the OAuth terminology to use in code, UI copy, and commits — e.g. **Authorization Server** (this service), **OAuth Client** (external app), **Consent**, **Scope**, **Authorization Grant/Denial**. Follow its "avoid" list (e.g. don't say "permission" for a protocol Scope).

## Backend / Frontend split

- **Backend** (REST API): `main.go`, `cmd/`, `internal/app/`, `internal/configs/`, `internal/container/`, `internal/entities/`, `internal/enums/`, `internal/common/`, `internal/v1/`, `pkg/`.
- **Frontend** (server-rendered UI): `internal/web/` — see its own section below.

## File naming conventions (important)

Files are split per domain, one route per file, but multiple related structs/functions may share a file when they belong to the same domain. Match existing suffixes exactly:

| Layer          | Directory                          | Suffix           | Example                |
|----------------|------------------------------------|------------------|------------------------|
| Entity         | `internal/entities/`               | `.entity.go`     | `user.entity.go`       |
| Model (DTO)    | `internal/v1/models/`              | `.model.go`      | `user.model.go`        |
| Repository     | `internal/v1/repositories/`        | `.repo.go`       | `user.repo.go`         |
| Service        | `internal/v1/services/`            | `.service.go`    | `user.service.go`      |
| Handler        | `internal/v1/handlers/`            | `.handler.go`    | `user.handler.go`      |
| Route          | `internal/v1/routes/`              | `.route.go`      | `user.route.go`        |

- A new domain (e.g. `foo`) means: `foo.entity.go`, `foo.model.go`, `foo.repo.go`, `foo.service.go`, `foo.handler.go`, `foo.route.go`. All six are usually needed.
- A file like `user.model.go` may hold multiple related types (e.g. `CreateUserModel`, `UserPageModel`). Group by domain, not one-type-per-file.
- Do not put routes in a single `routes.go`; each resource gets its own `*.route.go` and is wired up in `routes/router.go`.

## Backend architecture

- **Stack**: Go 1.26, Gin, GORM + PostgreSQL, JWT (golang-jwt/v5) + argon2id passwords, Cobra CLI, Swagger via swaggo, n8n integration.
- **Entry point**: `main.go` → `cmd.Execute()` (Cobra). Commands: `serve`, `migrate` (`--drop` to drop first).
- **Dependency injection**: `go.uber.org/dig`. Register every new repo/service/handler in `internal/container/di.go`. Repos and services are registered as interfaces via `dig.As(new(repo.IFooRepo))`. The `App` struct in `internal/app/app.go` receives the top-level handler groups (`routes.APIHandlers`, `webroutes.WebHandlers`); when adding a new domain you must: add it to the handler group struct, register in `di.go`, and wire its routes in the relevant `router.go`.
- **Generic CRUD contract**: repos implement `common.IBaseCrudRepo[TEntity, TInput, TPage]` and services implement `common.IBaseCrudService[...]` (see `internal/common/base_crud.go`). Extend the interface for domain-specific methods (e.g. `Login`, `GetPermissions`).
- **`Read` is pagination-driven via `morkid/paginate`**: the `Read` repo/service methods take a `*gin.Context` and call `paginate.New().With(stmt).Request(c.Request)` to parse pagination query params straight from the HTTP request. Therefore, when the web UI calls a `Read` service, pagination must be carried by the **browser URL query string** (e.g. HTMX `hx-get` links with the page params in the URL) — do not construct query params programmatically inside the handler.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarcelArt/oauth2-impl](https://github.com/MarcelArt/oauth2-impl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
