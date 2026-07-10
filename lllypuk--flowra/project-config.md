---
trigger: always_on
description: `AGENTS.md` is the single source of contributor guidance for this repository.
---

# Repository Guidelines

## Context Requirements
`AGENTS.md` is the single source of contributor guidance for this repository.

## Project Overview

Flowra is a chat-centric collaboration system with integrated task management and help desk capabilities.

Core stack:
- Backend: Go 1.26+ with Echo v4.
- Data: MongoDB 6+ (Go Driver v2) + Redis.
- Frontend: HTMX 2+, Pico CSS v2, vanilla JS modules.
- Auth: Keycloak SSO.
- Runtime: Docker Compose for local/dev and self-hosted helpers.

## Project Structure & Module Organization
`flowra` is a Go monorepo using clean architecture with event sourcing:

- **`cmd/`**: Application entry points (`api`, `worker`, `tools`)
- **`internal/`**: Private application code organized by layer:
  - `domain/`: Aggregates, entities, events, domain logic (6 aggregates, 30+ events)
  - `application/`: Use cases and business workflows (40+ use cases)
  - `infrastructure/`: External dependencies (MongoDB, Redis, Keycloak, EventStore)
  - `handler/http/`: HTTP request handlers (REST + HTMX endpoints)
  - `handler/websocket/`: WebSocket handlers for real-time updates
  - `middleware/`: HTTP middleware (auth, CORS, logging, rate limiting)
  - `service/`: Business services (workspace access, chat, member, auth)
  - `worker/`: Background workers (user sync)
- **`web/`**: Frontend (HTMX + Pico CSS templates, static assets)
- **`tests/`**: Test suites organized by scope:
  - `integration/`: Integration tests with real infrastructure (testcontainers)
  - `e2e/`: End-to-end tests (API and frontend browser tests)
  - `e2e/frontend/`: Playwright-based browser E2E tests
  - `load/`: Manual load tests (k6 scripts)
  - `mocks/`: Shared mock implementations
  - `testutil/`: Test utilities and helpers
- **`configs/`**: Configuration files
- **`docs/`**: Documentation (architecture, API specs, guides)

## Build, Test, and Development Commands

### Development
```bash
# Start full development environment (recommended)
make dev                    # Docker infra + worker + API

# Start API only (no worker, limited features)
make dev-lite              # FLOWRA_DEV_MODE=lite go run ./cmd/api

# Build binaries
make build                 # Creates bin/api and bin/worker

# Manage infrastructure
make docker-up            # Start MongoDB, Redis, Keycloak
make docker-down          # Stop all services
make reset-data           # Reset Chat=SoT data (when switching branches)
```

### Testing
```bash
# Run all tests
make test                             # Full suite with race detector

# Run specific test types
make test-unit                        # Unit tests: go test ./internal/...
make test-integration                 # Integration: -tags=integration
make test-e2e                         # E2E API: -tags=e2e
make test-e2e-frontend                # Browser E2E: -tags=e2e ./tests/e2e/frontend/...
make test-e2e-frontend-smoke          # Quick smoke test for board/sidebar

# Run a single test
go test -v ./internal/domain/chat -run TestChat_NewChat
go test -tags=integration -v ./tests/integration -run TestChatSoT

# Generate coverage
make test-coverage                    # Generates coverage.html

# Load testing (manual, requires k6 and AUTH_TOKEN)
make test-load-tags
```

### Code Quality
```bash
# Format and lint (ALWAYS run before committing)
make lint                  # go fmt + golangci-lint --fix

# Dependencies
make deps                  # go mod download && go mod tidy

# Playwright setup (for frontend E2E)
make playwright-install    # Install Chromium browser

# Self-hosted/production helpers
make docker-build          # Build production image
make docker-prod-up        # Start self-hosted production stack
make docker-prod-logs      # Tail production stack logs
make docker-prod-down      # Stop production stack
```

## Runtime & Access

- Main local app URL: `http://localhost:8080`
- API docs: `http://localhost:8080/docs`
- Health: `http://localhost:8080/health`
- Keycloak admin: `http://localhost:8090` (`admin/admin123`)
- Test user login: `testuser` / `test123`

Runtime toggles for API binary:
- `FLOWRA_WORKER=true` enables unified API + worker loops.
- `FLOWRA_WORKER=false` runs API-only mode.
- `--with-worker` / `--with-worker=false` overrides env value.

When switching branches around Chat=SoT changes, run `make reset-data` after bringing infra up.

## Configuration & Indexes

- Main configs: `configs/config.yaml`, `configs/config.dev.yaml`, `configs/config.prod.yaml`
- Local runtime stack: `docker-compose.yml`
- Self-hosted stack: `docker-compose.prod.yml`
- MongoDB indexes are managed in code at `internal/infrastructure/mongodb/indexes.go`
- Indexes are created on API startup and in integration test setup (no standalone migration runner required)

## Chat=SoT Guardrails

For typed entities (`task`, `bug`, `epic`) follow ADR-007 (`docs/architecture/adr-007-chat-sot.md`):
- Do not add new write handlers/commands under `internal/application/task`.
- Do not emit new `task.*` business write events.
- Typed entity writes must emit `chat.*` events only.
- Keep compatibility adapters on read/query side only.
- Keep assignee writes validated against real users.
- Do not re-introduce `TaskResult.Events`; task side effects are handled centrally in services.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lllypuk/flowra](https://github.com/lllypuk/flowra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
