---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
make build                  # Build binary (main.exe)
make run                    # Run locally (go run ./cmd/api)
make test                   # Run all tests (go test ./... -v)
make test-integration       # Integration tests (requires Docker, -tags=integration)
make lint                   # golangci-lint v2 (config: .golangci.yml)
make docker-run             # Docker Compose up (API + PostgreSQL + Redis + MinIO)
make docker-down            # Docker Compose down
make sqlc-generate          # Regenerate sqlc code in internal/sqlc/
make swagger                # Regenerate Swagger docs (swag init)
make seed                   # Seed admin user (go run ./cmd/seed)
make migrate-create name=x  # Create new migration pair
make watch                  # Live reload with Air
```

Run a single test file: `go test -v -run TestFunctionName ./internal/service/`

Production build: `CGO_ENABLED=0 go build -ldflags="-s -w" -trimpath -o server ./cmd/api`

## Architecture

**Request flow**: `Client → Middleware → Handler → Service → Repository → sqlc → PostgreSQL`

Each layer depends only on the layer below via **interfaces** (constructor injection):

- **Handler** (`internal/handler/`): Parse HTTP request, validate, call service, return `pkg/response` JSON. Swagger annotations on each method.
- **Service** (`internal/service/`): Business logic. Accepts repository/cache/email interfaces. Returns `*apperror.AppError` or domain errors.
- **Repository** (`internal/repository/`): Wraps sqlc-generated code. Translates `pgx.ErrNoRows` → `apperror.ErrNotFound`. Each repo accepts `sqlc.DBTX` (works with both `*pgxpool.Pool` and `pgx.Tx`).
- **sqlc** (`internal/sqlc/`): **DO NOT EDIT** — generated from `queries/*.sql` + `migrations/*.sql` via `make sqlc-generate`. Config: `sqlc.yaml`.

**Dependency injection** wired manually in `cmd/api/main.go` — no DI framework.

## Error Handling

- Return `*apperror.AppError` from services/handlers — auto-rendered as RFC 9457 Problem Details (`application/problem+json`) by `apperror.FiberErrorHandler`.
- Constructors: `NewBadRequest`, `NewUnauthorized`, `NewForbidden`, `NewNotFound`, `NewConflict`, `NewInternal`, `NewValidation(detail, []FieldError)`.
- Problem fields: `type` (from `ERROR_DOCS_BASE_URL` + `/errors/<kebab-code>`), `title`, `status`, `code` (snake_case i18n key), `detail`, `instance`, `requestId`, `timestamp`, `errors[]` (validation/business).
- Sentinel: `apperror.ErrNotFound` — repositories return this for missing records, services check with `errors.Is(err, apperror.ErrNotFound)`.

## Response Format

Use `pkg/response` — never write raw JSON in handlers:
- `response.Success(c, data)` — 200, resource returned directly (no envelope)
- `response.Created(c, data)` — 201, resource directly
- `response.NoContent(c)` — 204
- `response.List(c, data, hasMore)` — 200 Stripe-style `{object:"list", url, data, hasMore}`

JSON keys are camelCase; the correlation header is `X-Request-Id`.

## Key Patterns

### Config
`config/config.go` — struct-based config parsed from env vars via `caarlos0/env`. Loaded once in main, passed by pointer. See `.env.example` for all options.

### Validation
`pkg/validator` wraps `go-playground/validator`. Custom `password` tag: 8–72 chars (bcrypt limit), must include upper + lower + digit + special.

### Roles
Constants in `internal/dto/role.go`: `dto.RoleUser`, `dto.RoleAdmin`. Use these instead of magic strings.

### Pagination
List endpoints use forward-only cursor pagination (`dto.CursorQuery`: `limit` + `startingAfter`). Services call `buildCursor()`, repos expose `*Cursor` query methods backed by `(created_at, id)` indexes, handlers return `response.List(c, data, hasMore)`. Cursor helpers: `pagination.EncodeCursor/DecodeCursor/NormalizeLimit/RowLimit`. Offset helpers (`Normalize`, `LimitOffset`, `TotalPages`) remain for small tables on demand.

### Transactions
`pkg/database.TxManager.WithTx(ctx, func(tx pgx.Tx) error { ... })` — pass `tx` to repository constructors inside the callback.

### Pluggable Drivers
Storage (`pkg/storage`), Cache (`pkg/cache`), Email (`pkg/email`) — each has an interface and factory function (`NewStorage`, `NewCache`, `NewSender`) that switches on config driver string (`local`/`s3`/`minio`, `memory`/`redis`, `console`/`smtp`).

### JWT
`pkg/token` — `Generate(userID, role, secret, expireHour)` and `Parse(tokenStr, secret)`. Includes `iss`/`aud` claims for cross-service protection.

### Safe Int Conversion
Use `pagination.clampInt32()` for `int → int32` casts. Never cast directly — gosec G115 is enabled globally to catch unsafe conversions.

### Rate Limiting
Tiered rate limiters in `internal/router/v1.go`: `strictLimiter` (auth endpoints), `normalLimiter` (mutations), `relaxedLimiter` (reads). Configured via `RATE_LIMIT_*` env vars.

### Soft Delete
Users and files use soft delete (`deleted_at` column). Partial indexes (`WHERE deleted_at IS NULL`) on frequently queried columns.

### Lint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chuanghiduoc/fiber-golang-boilerplate](https://github.com/chuanghiduoc/fiber-golang-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
