---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UnifiedVoice is an enterprise event-driven speech processing platform: 19 Go microservices + Next.js frontend. Batch STT pipeline (upload → preprocess → transcribe → enrich), real-time streaming STT, TTS, speaker diarization, language detection, translation, with webhooks, billing, analytics, and admin controls. Deployed on Kubernetes (Kind for dev).

## Commands

### Local Development

```bash
make setup              # First-time: generate JWT keys, docker compose up, run migrations
make dev                # Start infrastructure (Postgres, Redis, Kafka, MinIO)
make dev-down           # Stop infrastructure
make run-auth           # Run a service locally (replace auth with any service name)
make run-orchestrator   # Services: auth, ingestion, orchestrator, preprocessing, stt-engine,
                        # transcript, langdetect, diarization, translation, notification,
                        # streaming-stt, billing, analytics, admin, tts, llm-enrichment, storage
```

Frontend: `cd web && npm install && npm run dev` (http://localhost:3000)

### Build & Test

```bash
make build              # Compile all Go binaries to bin/
make test               # Unit tests (all packages + services)
make lint               # golangci-lint
make proto              # Regenerate protobuf Go code from proto/

# E2E tests (80+ tests, requires services running)
make e2e-test           # Against local services
make e2e-scale          # Scale test: 100 concurrent jobs (timeout 600s)

# Run a single test
go test -v -run TestAuth ./tests/e2e/
go test -v -run TestHealthChecks ./tests/e2e/
```

### Kubernetes (Kind)

```bash
make kind-up            # Create 3-node Kind cluster
make kind-deploy        # Build images, load into Kind, deploy all manifests
make kind-status        # Check pods/services
make kind-logs SVC=auth # Tail logs for a service
make kind-test          # Port-forward + E2E tests against Kind
make kind-down          # Delete cluster
```

### Database

```bash
make migrate            # Run pending migrations for all services
make migrate-down       # Rollback 1 migration step per service
```

Migrations live at `migrations/<service>/` using golang-migrate format (`NNN_description.up.sql`, `NNN_description.down.sql`).

## Architecture

### Service Layout

Every Go service follows the same internal structure:

```
services/<name>/
  cmd/server/main.go          # Config load → DB → Redis → Kafka → Gin routes → graceful shutdown
  internal/
    domain/models.go          # Structs with json/db tags, string-typed enums
    handler/http_handler.go   # Gin routes, auth middleware, request binding
    service/<name>_service.go # Business logic, injected dependencies
    repository/postgres_*.go  # pgxpool queries, JSONB marshal/unmarshal
    consumer/consumer.go      # Kafka consumer (if event-driven)
  config.yaml                 # Local dev configuration
  Dockerfile                  # Multi-stage: golang:1.23-alpine → distroless
  go.mod                      # Module: github.com/unifiedvoice/services/<name>
```

### Shared Packages (`pkg/`)

All services import from these shared packages:

| Package | Purpose |
|---------|---------|
| `pkg/config` | `MustLoad[T](envPrefix, yamlPath)` — Viper-based typed config |
| `pkg/database` | `NewPool()` → `*pgxpool.Pool`, `Migrate()` |
| `pkg/auth` | JWT RS256 validation, RBAC, Gin middleware, `FromGinContext()` |
| `pkg/errors` | Structured `AppError` with codes (`NOT_FOUND`, `UNAUTHORIZED`, etc.) |
| `pkg/httputil` | Response envelope (`OK`, `Created`, `Error`), rate limiting, request binding |
| `pkg/kafka` | Consumer/Producer with protobuf serialization |
| `pkg/cache` | Redis client wrapper |
| `pkg/observability` | Zap logging + OpenTelemetry tracing setup |
| `pkg/health` | Liveness/readiness probe handlers |

### Event-Driven Pipeline

Kafka topic naming:
- **Commands**: `dispatch.<service>` (orchestrator → workers)
- **Events**: `<service>.completed`, `<service>.failed` (workers → orchestrator)
- **Lifecycle**: `jobs.lifecycle` (orchestrator → billing/analytics/notification)
- **Job creation**: `ingestion.jobs.created` (ingestion → orchestrator)

The orchestrator is a stateless event router: receives job.created → creates pipeline → dispatches steps sequentially via Kafka → each worker processes and publishes completed/failed → orchestrator advances to next step.

### Database per Service

Each service owns its PostgreSQL database (`unifiedvoice_<service>`). No cross-service queries. Databases initialized by `scripts/init-databases.sql`. Conventions:
- ULID primary keys (`VARCHAR(26)`)
- `TIMESTAMPTZ` for all timestamps
- `JSONB` for complex nested data
- `update_updated_at()` trigger on tables with `updated_at`
- Tenant isolation: `tenant_id VARCHAR(64)` on all tenant-scoped tables

### Frontend (`web/`)

Next.js 16 with App Router. Two route groups: `(auth)` for login/register, `(portal)` for authenticated pages.

- **State**: Zustand stores (`src/stores/`) + TanStack Query 5 for server state
- **API client**: Axios with JWT interceptor (`src/lib/api/client.ts`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kapiljain1989/unifiedVoice](https://github.com/kapiljain1989/unifiedVoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
