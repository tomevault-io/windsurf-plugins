---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Backend (Go)
```bash
make build          # Build frontend + Go binary (version-injected)
make build-go       # Build Go binary only
make dev-backend    # Run Go backend with dev tag (hot reload not included)
make test           # go test ./...
make lint           # golangci-lint run ./...
make tidy           # go mod tidy
make generate       # Regenerate all mocks via mockery (reads .mockery.yaml)
```

Run a single Go test:
```bash
go test ./internal/service/... -run TestChatService
```

### Frontend (React/TypeScript)
```bash
cd frontend && npm ci        # Install dependencies
make dev-frontend            # Vite dev server on :5173
npm run build                # TypeScript check + Vite bundle
npm run lint                 # ESLint
npm run typecheck            # TypeScript strict check
npm run format               # Prettier
```

### Development Setup
Two terminals are needed in dev mode:
1. `make dev-backend` — Go API server on `:8990` (or `PORT` env)
2. `make dev-frontend` — Vite dev server on `:5173` (proxies API calls to `:8990`)

### Required Environment
```bash
ANTHROPIC_API_KEY=...        # Optional (uses Claude Code CLI auth if unset)
AGENTO_DATA_DIR=~/.agento   # Optional, default: ~/.agento (supports ~ expansion, e.g. ~/.agento-dev for local dev)
PORT=8990                    # Optional, default: 8990
# OpenTelemetry (all optional — can also be configured via Settings UI)
OTEL_EXPORTER_OTLP_ENDPOINT=localhost:4317  # OTLP collector gRPC endpoint
OTEL_EXPORTER_OTLP_INSECURE=true           # Skip TLS for local collectors
OTEL_METRICS_EXPORTER=otlp                 # "otlp" or "prometheus"
OTEL_LOGS_EXPORTER=otlp                    # "otlp"
OTEL_EXPORTER_OTLP_HEADERS=key=value       # Auth headers for the collector
OTEL_METRIC_EXPORT_INTERVAL=60000          # Push interval in ms (default: 60000)
OTEL_SDK_DISABLED=true                     # Disable telemetry entirely
```

## Architecture

### Request Flow
```
Browser → Vite (dev) / embedded FS (prod) → React SPA
                                          ↓
                              chi router (internal/server/)
                                          ↓
                              API handlers (internal/api/)
                                          ↓
                              Services (internal/service/)
                                          ↓
                        Storage (internal/storage/) + Agent SDK
```

### Backend Layers

**`cmd/`** — Cobra CLI commands: `web` (HTTP server), `ask` (CLI), `update` (self-update). `cmd/assets.go` embeds the frontend build; `cmd/assets_dev.go` proxies to Vite.

**`internal/server/`** — Chi router setup with middleware (Recoverer, RequestID, request logger), wrapped in `otelhttp.NewHandler` for automatic HTTP trace/span generation. Mounts `/api` routes, serves SPA, and exposes a dynamic `/metrics` handler (Prometheus pull endpoint, active only when configured). Graceful shutdown with 5s timeout.

**`internal/api/`** — HTTP handlers. `Server` struct holds all service dependencies (including `MonitoringManager` and `AppConfig`). `Mount()` registers all routes. SSE streaming for live sessions via `livesessions.go` (includes per-session mutex for concurrent send serialization). `monitoring.go` exposes `GET/PUT /api/monitoring` and `POST /api/monitoring/test` for OTel configuration. `PATCH /api/chats/{id}` and `PATCH /api/claude-sessions/{id}` support title editing and favorites. `types.go` defines request/response types shared across handlers. `uploads.go` handles `POST /api/uploads` for file/image drag-drop and paste (saves to `tmp-uploads/` directory). `claude_session_insights.go` and `insight_store_adapter.go` expose per-session and aggregated insight metrics. `claude_session_journey.go` provides a step-by-step timeline visualization endpoint.

**`internal/service/`** — Business logic. `ChatService`, `AgentService`, `IntegrationService`, `NotificationService`, `TaskService`, and `ClaudeSettingsProfileService` interfaces decouple handlers from storage. `errors.go` defines typed errors for HTTP mapping.

**`internal/agent/`** — Integration with `github.com/shaharia-lab/claude-agent-sdk-go`. `runner.go` converts agent config to SDK `RunOptions`, executes sessions, streams results. `tracing.go` provides OTel span helpers for per-tool-call and per-run tracing in both chat and scheduler paths.

**`internal/storage/`** — SQLite persistence (`~/.agento/agento.db`). `SQLiteAgentStore`, `SQLiteChatStore`, `SQLiteIntegrationStore`, `SQLiteSettingsStore`, `SQLiteNotificationStore`, `SQLiteTaskStore`, `SQLiteSessionInsightsStore` implement store interfaces. `migrate_fs_to_sqlite.go` handles one-time migration from the legacy filesystem format. `telemetry.go` provides `withStorageSpan` helper for OTel span/metric instrumentation on storage operations. Uses `modernc.org/sqlite` (pure Go, no CGo).

**`internal/config/`** — Shared configuration layer. `AppConfig` loads from env. `profiles.go` has shared profile types to prevent import cycles. **Import rule**: `config` ← `service` ← `api` (never reverse).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shaharia-lab/agento](https://github.com/shaharia-lab/agento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
