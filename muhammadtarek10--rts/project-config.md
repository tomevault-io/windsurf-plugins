---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

RTS is a microservices monorepo with a polyglot backend and a Vue 3 frontend:

| Service                   | Language/Framework     | Database                 | Host Port  |
| ------------------------- | ---------------------- | ------------------------ | ---------- |
| `services/auth`           | NestJS (TypeScript)    | PostgreSQL (Drizzle ORM) | 3001       |
| `services/catalog`        | ASP.NET Core (.NET 10) | MongoDB                  | 3002       |
| `services/inventory`      | Go (stdlib)            | PostgreSQL (go-migrate)  | 3004       |
| `services/orders`         | (planned)              | -                        | -          |
| `services/payment`        | (planned)              | -                        | -          |
| `services/notification`   | (planned)              | -                        | -          |
| `services/recommendation` | (planned)              | -                        | -          |
| `app`                     | Vue 3 + Vite + Pinia   | -                        | 80 (nginx) |

**Infrastructure (docker-compose.yml):** PostgreSQL (5432), MongoDB (27017), Redis (6379), RabbitMQ (5672/15672), MinIO (9000/9001), Nginx (80/443), Grafana (3000), Redis Insight (5540), Certbot.

**Environment:** All docker-compose environment values are sourced from the root `.env` file — no hardcoded values in `docker-compose.yml`.

## Auth Service (`services/auth`)

NestJS app with Passport.js strategies (local, JWT access token, JWT refresh token). Auth events are published to RabbitMQ for other services to consume.

**Module structure:** `core/` (config, database, broker, utils, decorators, filters, interceptors) + `auth/` + `users/`

**Migrations:** Drizzle Kit, migrations stored at `src/core/database/migrations/`.

```bash
# Dev
cd services/auth
npm install
npm run dev              # watch mode with NODE_ENV=development

# Migrations (dev)
npm run dev:migrate      # generate + migrate using .env.development

# Tests
npm run test             # unit tests
npm run test:e2e         # e2e tests
npm run test:cov         # coverage

# Local dev DB
docker compose up -d     # starts auth-database (postgres:18 on port 5439)

# Drizzle Studio
npm run dev:studio
```

## Catalog Service (`services/catalog`)

ASP.NET Core Web API (.NET 10) using Clean Architecture layers: `Domain/`, `Application/`, `Infrastructure/`, `Controllers/`. JWT bearer auth is validated using the shared secret from the auth service.

**API conventions:** JSON responses use `snake_case` field naming (configured via `JsonNamingPolicy.SnakeCaseLower` in `Program.cs`). Internal serialization (RabbitMQ events, Redis cache) uses `camelCase` — do not change those.

**Local dev uses port 27020 for MongoDB** (via `services/catalog/docker-compose.yml`), while the root docker-compose uses 27017.

```bash
# Dev
cd services/catalog
docker compose up -d            # starts MongoDB on port 27020

cd Catalog.Api
dotnet run                      # starts the API (Swagger at /swagger in dev)
dotnet test                     # run tests (solution-level)
dotnet format Catalog.slnx      # format C# code
```

## Inventory Service (`services/inventory`)

Go service using only the standard library (`net/http`, `database/sql`). Manages warehouses, stock levels, reservations, and stock movements. Consumes catalog events from RabbitMQ to auto-create inventory items when products are created.

**Architecture:** `cmd/server/` (entrypoint + bootstrap) + `internal/` (domain, handler, service, repository, consumer, publisher, middleware, cache, router).

**API conventions:** JSON responses use `snake_case` field naming. JWT bearer auth validated using the shared secret from the auth service. Swagger UI at `/swagger/` (basic-auth protected).

**Concurrency:** Background goroutines for AMQP catalog consumer, reservation expiry sweeper (30s tick), and graceful shutdown handler. `sync.Mutex` guards the AMQP publisher channel.

**Migrations:** `golang-migrate` with SQL files in `migrations/`. Runs automatically on startup.

```bash
# Dev
cd services/inventory
docker compose up -d            # starts PostgreSQL on port 5433
go run ./cmd/server             # starts the API on :8080

# Tests
go test ./...                   # all tests
go test ./internal/repository/... -tags=integration  # integration tests (needs DB)

# Swagger
make swagger                    # regenerate docs (requires swag CLI)

# Format
gofumpt -w .
```

## Frontend (`app`)

Vue 3 + Vite + Pinia (with persisted state) + Vue Router. Written in TypeScript, styled with LESS.

```bash
cd app
npm install
npm run dev      # vite dev server
npm run build    # vue-tsc + vite build
```

## Full Stack (docker-compose)

```bash
# Start all infrastructure + services
docker compose up -d

# Start individual infrastructure
docker compose up -d db mongo redis rabbitmq

# Build and start a specific service
docker compose up -d --build catalog
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MuhammadTarek10) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
