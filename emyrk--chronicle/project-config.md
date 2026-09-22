---
trigger: always_on
description: You are an experienced, pragmatic software engineering AI agent. Do not over-engineer a solution when a simple one is possible. Keep edits minimal. If you want an exception to ANY rule, you MUST stop and get permission first.
---

You are an experienced, pragmatic software engineering AI agent. Do not over-engineer a solution when a simple one is possible. Keep edits minimal. If you want an exception to ANY rule, you MUST stop and get permission first.

# Chronicle – Agent Contribution Guide

## Project Overview

Chronicle is a **game-play performance analysis tool for Classic World of Warcraft**, specifically targeting the Turtle WoW server. It transforms combat logs into accessible insights for raid leaders, tracking damage, healing, and other raid metrics.

### Technology Stack

| Layer     | Technology                                     |
| --------- |------------------------------------------------|
| Backend   | Go 1.25+                                       |
| Frontend  | React 19 + TypeScript + Vite + Tailwind CSS v4 |
| Database  | PostgreSQL 17 (via pgx/v5)                     |
| ORM/Query | sqlc (code generation from SQL)                |
| Router    | go-chi/chi                                     |
| Auth      | OAuth (Discord) + JWT sessions                 |
| Queue     | River (PostgreSQL-based job queue)             |
| Package   | pnpm (frontend)                                |
| Hosting   | Railway                                        |

## Reference

### Important Directories

```
cmd/
├── chronicle/      # CLI tool
├── chronicled/     # Main server daemon
└── wasm/           # WebAssembly build for browser parsing

api/
├── api.go          # API router and initialization
├── chronauth/      # Authentication (OAuth, sessions, JWT)
├── chroniclesdk/   # SDK types (used for TypeScript generation)
├── httpapi/        # HTTP response helpers (Write, Read, InternalServerError)
├── httpmw/         # HTTP middleware (auth, recovery, prometheus)
└── db2sdk/         # Database model to SDK conversion

chronicle/          # Core business logic (log parsing queue, uploads)
combatlog/          # Combat log parsing engine (vanilla WoW format)
database/
├── migrations/     # SQL migration files (numbered)
├── queries/        # Raw SQL queries for sqlc
├── dbtestutil/     # Test helpers (Postgres in Docker)
├── pubsub/         # PostgreSQL LISTEN/NOTIFY wrapper
└── storage/        # Object storage interface

frontend/chronicle/ # React frontend (Vite project)
internal/           # Shared utilities (testutil, cryptorand, etc.)
```

### Key Files

- `api/api.go` – API routes definition, middleware setup
- `chronicle/chronicle.go` – Core Chronicle service (uploads, parsing)
- `database/sqlc.yaml` – sqlc configuration
- `database/generate.sh` – Custom sqlc output merging script
- `Makefile` – Primary build/dev commands

## Essential Commands

### Build

```bash
# Build full project (backend + frontend)
make build

# Build backend only
make build-backend

# Build frontend only
cd frontend/chronicle && pnpm build
```

### Development

```bash
# Start local dependencies first (Postgres on :5433, SpiceDB, OCR)
make services-up

# Start full dev server (backend on :4000, builds frontend)
make develop

# Start backend only without requiring frontend/dist assets
make develop-backend

# Start frontend dev server with hot reload (proxies to backend)
cd frontend/chronicle && pnpm dev
```

### Test

```bash
# Run all tests (requires Postgres, use docker or local)
make test

# Start Postgres in Docker for testing
make test-postgres-docker

# Create the chronicle database when using local Postgres client tools
make create-db
```

### Lint

```bash
# Go linting
make lint
# or directly:
golangci-lint run

# Frontend linting
cd frontend/chronicle && pnpm lint
```

### Code Generation

```bash
# Regenerate everything (database, WASM, TypeScript types)
make gen

# Database only (after changing migrations or queries)
make gen/db

# TypeScript API types from Go SDK
# (auto-runs via make gen)
go run -C ./scripts/apitypings main.go > frontend/chronicle/src/api/typesGenerated.ts
```

### Database Migrations

**Migrations are immutable once deployed.** Never edit an existing migration file — always create a new one. If you are unsure whether a migration has been deployed, ask the user before modifying it.

> [!CAUTION]
> **Production migrations run during service startup and block the service from becoming available.** Before adding a data migration that updates, deletes, rewrites, or scans an existing table, inspect production-scale row counts and relevant indexes, estimate the affected rows/WAL, and explicitly warn the user if it may delay startup or cause downtime. Get the user's approval before shipping a potentially slow migration.
>
> Avoid broad updates of large denormalized/history tables such as `ranking_snapshot_members`, `encounter_dps_rankings`, and `parse_score_results`. Scope updates to the exact cohort, use indexed predicates, and prefer batching, an offline/admin backfill, or recomputation when a startup transaction could touch a large number of rows. Include a preview/count query with the proposed migration so impact can be checked before deployment.

```bash
# Create a new migration
./database/migrations/create_migration.sh "description of migration"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Emyrk/chronicle](https://github.com/Emyrk/chronicle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
