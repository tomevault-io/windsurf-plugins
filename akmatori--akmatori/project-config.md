---
trigger: always_on
description: Akmatori is an AI-powered AIOps platform that receives alerts from monitoring systems (Zabbix, Alertmanager, PagerDuty, Grafana, Datadog), analyzes them using multi-provider LLM agents (via the pi-mono coding-agent SDK), and executes automated remediation.
---

# Claude Code Instructions for Akmatori

## Project Overview

Akmatori is an AI-powered AIOps platform that receives alerts from monitoring systems (Zabbix, Alertmanager, PagerDuty, Grafana, Datadog), analyzes them using multi-provider LLM agents (via the pi-mono coding-agent SDK), and executes automated remediation.

## Architecture

- **5-container Docker architecture**: API, Agent Worker, MCP Gateway, PostgreSQL, QMD (runbook search)
- **Backend**: Go 1.24+ (API server, MCP gateway)
- **Agent Worker**: Node.js 22+ / TypeScript using `@mariozechner/pi-coding-agent` SDK (v0.67.6)
- **Frontend**: React 19 + TypeScript + Vite + Tailwind
- **Database**: PostgreSQL 16 with GORM
- **LLM Providers**: Anthropic, OpenAI, Google, OpenRouter, Custom (configured via web UI)

## Key Directories

```
/opt/akmatori/
├── cmd/akmatori/           # Main API server entry point
├── internal/
│   ├── alerts/adapters/    # Alert source adapters (Zabbix, Alertmanager, etc.)
│   ├── alerts/extraction/  # AI-powered alert extraction from free-form text
│   ├── api/                # Request/response helpers, pagination
│   ├── database/           # GORM models and database logic
│   ├── handlers/           # HTTP/WebSocket handlers
│   ├── middleware/         # Auth, CORS middleware
│   ├── output/             # Agent output parsing (structured blocks)
│   ├── logging/           # Structured logging (slog) initialization
│   ├── services/           # Business logic layer (+ interfaces.go for testability)
│   ├── setup/              # Zero-config first-run setup
│   ├── slack/              # Slack integration (Socket Mode, hot-reload)
│   ├── testhelpers/        # Test utilities, builders, mocks
│   └── utils/              # Utility functions
├── agent-worker/           # Node.js/TypeScript agent worker
│   └── src/                # TypeScript source (gateway-client, gateway-tools, script-executor)
├── mcp-gateway/            # MCP protocol gateway (separate Go module)
│   └── internal/
│       ├── auth/           # Per-incident tool authorization (allowlist enforcement)
│       ├── cache/          # Generic TTL cache
│       ├── mcpproxy/       # MCP proxy: connection pool + handler for external MCP servers
│       ├── ratelimit/      # Token bucket rate limiter
│       └── tools/          # SSH, Zabbix, VictoriaMetrics, PostgreSQL, ClickHouse, Grafana, Catchpoint, PagerDuty, NetBox, Kubernetes, and HTTP connector implementations
├── web/                    # React frontend
├── qmd/                    # QMD search sidecar (Dockerfile, config, entrypoint)
├── docs/                   # OpenAPI specs (swagger at /api/docs)
└── tests/fixtures/         # Test payloads and mock data
```

## CRITICAL: Always Verify Changes with Tests

**After ANY code change, run the appropriate test command:**

| After changing... | Run command |
|-------------------|-------------|
| Alert adapters (`internal/alerts/adapters/`) | `make test-adapters` |
| MCP Gateway (`mcp-gateway/`) | `make test-mcp` |
| Agent worker (`agent-worker/`) | `make test-agent` |
| Any Go code | `make test` |
| Before committing | `make verify` |

```bash
# Quick reference
make test-adapters    # ~0.01s
make test-mcp         # ~0.01s
make test-all         # All tests including agent-worker
make verify           # go vet + all tests (pre-commit)
```

## CRITICAL: Rebuild Docker Containers After Changes

| After changing... | Rebuild command |
|-------------------|-----------------|
| API server (`cmd/`, `internal/`) | `docker-compose build akmatori-api && docker-compose up -d akmatori-api` |
| MCP Gateway (`mcp-gateway/`) | `docker-compose build mcp-gateway && docker-compose up -d mcp-gateway` |
| Agent worker (`agent-worker/`) | `docker-compose build akmatori-agent && docker-compose up -d akmatori-agent` |
| Frontend (`web/`) | `docker-compose build frontend && docker-compose up -d frontend` |
| QMD search (`qmd/`) | `docker-compose build qmd && docker-compose up -d qmd` |

## Current Testing Priorities

Coverage moves quickly; re-run `go test -coverprofile=coverage.out ./...` before quoting numbers.
Focus new tests on historically weak areas: `internal/handlers`, `internal/services`, `internal/slack`, main-module database paths, and MCP Gateway `internal/tools` / `internal/tools/zabbix`.

## Recent Changes (Apr 2026)

- Slack skill launches now start fresh agent sessions; skill prompt trimming/output-format handling was tightened.
- Alert webhook error paths, setup-state edge cases, and Slack settings integration helpers all gained focused tests.
- When updating docs, prefer small pattern notes over long examples so this file stays under the 30k limit.

## Agent Worker Architecture

The `agent-worker/` uses `@mariozechner/pi-coding-agent` SDK (v0.67.6):

| Component | File | Purpose |
|-----------|------|---------|
| Entry Point | `src/index.ts` | Reads config, starts orchestrator |
| Orchestrator | `src/orchestrator.ts` | Routes WebSocket messages |
| Agent Runner | `src/agent-runner.ts` | Creates pi-mono sessions |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akmatori/akmatori](https://github.com/akmatori/akmatori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
