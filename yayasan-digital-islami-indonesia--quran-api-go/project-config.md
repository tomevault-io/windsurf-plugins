---
trigger: always_on
description: This file is the authoritative guide for any AI agent working on this codebase. Read it fully before making any changes. Do not deviate from the rules below.
---

# agents.md — Quran API Go

This file is the authoritative guide for any AI agent working on this codebase. Read it fully before making any changes. Do not deviate from the rules below.

---

## Project Summary

An internal RESTful API that serves Al-Quran data (Arabic text, Indonesian & English translations) for the Ilmunara super app. This is **not a public API**. It is a lightweight internal data service.

**Primary consumers:** Ilmunara super app (internal only).

---

## Non-Negotiable Constraints

These are governance-level rules. Never violate them, even if a prompt asks you to.

- **No microservices.** This is a monolith modular architecture. Do not split into separate services.
- **No Redis.** Not for rate limiting, not for caching. Removed by policy for MVP.
- **No DI framework.** No Wire, no Uber FX. Use manual constructor injection only.
- **No rate limiting middleware.** Out of scope for MVP. Do not add it.
- **No authentication middleware.** Out of scope for MVP. Do not add it.
- **No write endpoints.** The database is read-only after seeding. Never add POST, PUT, PATCH, or DELETE handlers.
- **No new dependencies without justification.** If a task can be done with the standard library or existing dependencies, use those. Do not add new `go.mod` entries speculatively.
- **No wildcard CORS.** `Allow-Origin` must use the `ALLOWED_ORIGINS` env variable. Never hardcode `*`.

---

## Tech Stack

| Layer | Choice |
|-------|--------|
| Language | Go 1.22+ |
| HTTP Framework | Gin |
| Database | SQLite via `modernc.org/sqlite` (pure Go, no CGO) |
| Migrations | Goose |
| Full-text Search | SQLite FTS5 |
| Logging | zerolog |
| Documentation | Scalar (OpenAPI 3.0) |

---

## Project Structure

```
quran-api-go/
├── cmd/
│   ├── api/
│   │   └── main.go           ← Manual DI wiring. All constructors called here.
│   ├── migrate/
│   │   └── main.go           ← Migration runner
│   └── seed/
│       └── main.go           ← Seed runner
├── internal/
│   ├── config/               ← Env loading only. No business logic.
│   ├── database/             ← SQLite connection wrapper
│   ├── domain/
│   │   ├── errors.go         ← Sentinel errors (ErrNotFound, etc.)
│   │   ├── surah/            ← surah entity, repository interface, service
│   │   ├── ayah/             ← ayah entity, repository interface, service
│   │   ├── juz/              ← juz entity, repository interface, service
│   │   └── search/           ← search entity, repository interface, service
│   ├── handler/              ← HTTP handlers. One file per domain.
│   ├── repository/           ← SQLite implementations of repository interfaces
│   ├── service/              ← Business logic. Orchestrates repositories.
│   └── middleware/
│       ├── cors.go
│       ├── logging.go
│       └── recovery.go
├── pkg/
│   ├── response/             ← Shared HTTP response helpers
│   ├── pagination/           ← Shared pagination parser
│   └── validator/            ← Shared input validators (lang, ID, range)
├── migrations/               ← Goose SQL migration files
├── scripts/seed/             ← Data seeder logic
├── docs/                     ← Documentation (openapi.yaml, etc.)
├── data/                     ← quran.db lives here
├── .env.example
├── Dockerfile
├── Makefile
└── go.mod
```

**Rules:**
- `internal/` is for application code. `pkg/` is for shared utilities with no business logic.
- Domain interfaces live in `internal/domain/<name>/repository.go` and `service.go`
- Domain entities live in `internal/domain/<name>/entity.go`
- SQLite implementations live in `internal/repository/` (not in domain)
- Never put business logic in `handler/`. Handlers only parse input, call service, and write response.
- Never put SQL queries in `service/`. SQL belongs in `repository/` only.
- Never import `handler/` from `service/` or `repository/`. Dependency flow is one-way: `handler → service → repository`.

---

## Dependency Injection Pattern

All wiring happens in `cmd/api/main.go`. No constructors auto-discover or register themselves.

**Correct:**
```go
// cmd/api/main.go
import (
    "quran-api-go/internal/database"
    "quran-api-go/internal/repository"
    "quran-api-go/internal/service"
    "quran-api-go/internal/handler"
    "quran-api-go/internal/domain/surah"
)

db := database.New(cfg.DBPath)

// Repository implementations (in internal/repository/)
surahRepo := repository.NewSurahRepository(db)

// Service implementations (in internal/service/)
surahService := service.NewSurahService(surahRepo)

// Handlers (in internal/handler/)
surahHandler := handler.NewSurahHandler(surahService)

r := gin.New()
r.GET("/surah", surahHandler.List)
r.GET("/surah/:id", surahHandler.Detail)
```

**Wrong — do not do this:**
```go
// Do not use any container, provider, or injector pattern
fx.New(
    fx.Provide(repository.NewSurahRepository),
    ...
)
```

---

## Naming Conventions

### Files
- One file per domain per layer: `surah_repository.go`, `surah_service.go`, `surah_handler.go`
- Middleware files are single-word: `cors.go`, `logging.go`, `recovery.go`
- Migration files follow Goose convention: `00001_init.sql`

### Interfaces

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yayasan-Digital-Islami-Indonesia/quran-api-go](https://github.com/Yayasan-Digital-Islami-Indonesia/quran-api-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
