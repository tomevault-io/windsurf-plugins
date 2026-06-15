---
trigger: always_on
description: **usulnet** is a self-hosted Docker Management Platform written in **Go** — a single-binary alternative to Portainer. It provides container lifecycle management, security scanning, backup management, reverse proxy configuration, monitoring, multi-node orchestration, and developer tooling through a unified web UI.
---

# GitHub Copilot Instructions for usulnet

## Project Overview

**usulnet** is a self-hosted Docker Management Platform written in **Go** — a single-binary alternative to Portainer. It provides container lifecycle management, security scanning, backup management, reverse proxy configuration, monitoring, multi-node orchestration, and developer tooling through a unified web UI.

- **Module path:** `github.com/fr4nsys/usulnet`
- **Go version:** 1.25.7
- **License:** AGPL-3.0-or-later
- **Current release:** v26.2.0 Beta

## Architecture

### Operation Modes

The platform supports three operation modes:

1. **standalone** — Single Docker host. All services local. No NATS required.
2. **master** — Standalone + NATS gateway server for remote agent management.
3. **agent** — Connects to master via NATS. No web UI. Executes Docker operations on its local host.

### Entry Points

| Binary | Location | Framework | Purpose |
|--------|----------|-----------|---------|
| `usulnet` | `cmd/usulnet/` | Cobra CLI | Main server (`serve`, `migrate`, `config`, `admin`, `version`) |
| `usulnet-agent` | `cmd/usulnet-agent/` | `flag` package | Remote node agent |

### Key Directories

- `cmd/` — Binary entry points (usulnet and usulnet-agent)
- `internal/api/` — REST API layer with Chi router and middleware
- `internal/services/` — Business logic layer (~39 service packages)
- `internal/repository/postgres/` — PostgreSQL repositories with migrations
- `internal/web/` — Web UI layer (server-side rendered with Templ)
- `web/static/` — Static assets (CSS, JS, fonts)
- `tests/` — Tests (unit, integration, e2e, benchmarks, load)

## Technology Stack

### Backend

- **Language:** Go 1.25.7
- **HTTP Router:** go-chi/chi/v5
- **Database:** PostgreSQL via pgx/v5 + sqlx
- **Cache:** Redis (go-redis/v9)
- **Messaging:** NATS with JetStream
- **Docker:** Docker SDK v28.5.1
- **Auth:** JWT (golang-jwt/jwt/v5), OIDC, LDAP
- **Logging:** uber-go/zap with structured logging
- **Config:** spf13/viper + spf13/cobra
- **Validation:** go-playground/validator/v10

### Frontend

- **Templates:** Templ v0.3.977 (type-safe Go HTML templates)
- **CSS:** Tailwind CSS v3.4.17 (standalone CLI, no Node.js)
- **Reactivity:** Alpine.js 3.14.8
- **Interactivity:** HTMX 2.0.4
- **Charts:** Chart.js
- **Terminal:** xterm.js (WebSocket-based)
- **Code Editor:** Monaco Editor
- **RDP:** Apache Guacamole
- **Fonts:** IBM Plex Sans/Mono, Space Grotesk (self-hosted)

## Code Conventions

### File Naming

- `handler_<feature>.go` — Web page handlers
- `adapter_<feature>.go` — Type adapters (service → web layer)
- `routes_<area>.go` — Route registration
- `*_templ.go` — Auto-generated from `.templ` files (DO NOT EDIT)
- `*_test.go` — Test files

### Required Copyright Header

Every Go file must include this header:

```go
// SPDX-License-Identifier: AGPL-3.0-or-later
// Copyright (c) 2024-2026 usulnet contributors
// https://github.com/fr4nsys/usulnet
```

### Import Organization

Group imports in this order:
1. Standard library
2. Third-party packages
3. Local packages (`github.com/fr4nsys/usulnet/...`)

Local imports use the module path prefix configured in `.golangci.yml` via goimports.

### Error Handling

- Always wrap errors with context: `fmt.Errorf("descriptive context: %w", err)`
- Services return errors; handlers log and respond with structured JSON
- Non-fatal failures are logged but don't block startup

### Logging

- Use **zap** via wrapper at `internal/pkg/logger`
- Structured key-value pairs: `log.Info("message", "key", value, "key2", value2)`
- Levels: debug, info, warn, error, fatal
- JSON in production, console in development

### Service Architecture

- **Constructor injection:** `NewService(repo, deps..., config, logger)`
- All services accept a `*logger.Logger`
- Lifecycle methods: `Start(ctx)` / `Stop()` where applicable
- **Nil-safe design:** handlers check if services are nil
- **Repository pattern:** each domain has a repository in `internal/repository/postgres/`

### RBAC

Three role tiers enforced via Chi middleware:
- `admin` — Full access (`RequireAdmin` middleware)
- `operator` — Operational access (`RequireOperator` middleware)
- `viewer` — Read-only (`RequireViewer` middleware)

## Build & Development

### Quick Commands

```bash
# Build
make build           # Full build: templ + CSS + go build
make build-agent     # Build agent binary only
make frontend        # Generate templates + compile CSS

# Run
make run             # go run ./cmd/usulnet
make dev-up          # Start dev services (PostgreSQL, Redis, NATS, MinIO)
make dev-down        # Stop dev services

# Test
make test            # go test -v -race -cover ./...
make test-coverage   # Coverage with HTML report
make test-e2e        # E2E tests (requires services, build tag: e2e)

# Quality
make lint            # golangci-lint run ./...
make lint-fix        # Auto-fix linting issues
make fmt             # gofmt -s -w .
make vet             # go vet ./...
make quality         # Full quality gate (lint + vet + coverage check)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fran-olivares/usulnet](https://github.com/fran-olivares/usulnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
