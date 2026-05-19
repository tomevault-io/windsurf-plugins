---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Getting Started

**Prerequisites** (choose one):
- **Native**: Go 1.24.2 or later
- **Docker**: Docker 20.10+ and Docker Compose v2.0+

**Setup**:
- Native: Copy `config.example.json` to `config.json` and configure required values
- Docker: Copy `config.docker.example.json` to `config.docker.json` and change `api-keys` to secure values

**Verify installation**: `http://localhost:4000/healthz`

## Development Commands

All commands are managed through the Makefile:

- `make run` - Build and run the server with config from `config.json`
- `make build` - Build the application binary to `bin/maglev`
- `make test` - Run all tests
- `make load-test` - Run smoketest and stresstest (k6)
- `make lint` - Run golangci-lint (requires: `go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest`)
- `make coverage` - Generate test coverage report with HTML output
- `make coverage-report` - Output per-package test coverage as JSON for CI parsing (requires jq)
- `make models` - Regenerate sqlc models from SQL queries
- `make watch` - Run with Air for live reloading during development
- `make fmt` - Format all Go code with `go fmt`
- `make clean` - Clean build artifacts
- `make build-pure` - Build without CGO (pure Go SQLite driver)
- `make test-pure` - Run tests without CGO
- `make update-openapi` - Fetch latest upstream OpenAPI spec and overwrite `testdata/openapi.yml`
- `make check-openapi` - Verify `testdata/openapi.yml` matches upstream (exits 1 if out of date)

**Build tags**: When running `go` commands directly (not via Makefile), you must pass `-tags "sqlite_fts5 sqlite_math_functions"` for CGO builds or `-tags "purego"` for pure Go builds.

**OpenAPI spec**: CI checks that `testdata/openapi.yml` is in sync with [OneBusAway/sdk-config](https://github.com/OneBusAway/sdk-config/blob/main/openapi.yml) on every push and PR. If upstream has changed, CI fails — run `make update-openapi` locally and commit the updated file.

## Load Testing and Profiling

See `loadtest/README.md`. Start with pprof enabled: `MAGLEV_ENABLE_PPROF=1 make run`, then run `k6 run loadtest/k6/scenarios.js`. Capture CPU profiles with `go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30`.

## Docker Commands

Docker provides a consistent development environment across all platforms:

- `make docker-build` - Build the Docker image
- `make docker-run` - Build and run the container with mounted config
- `make docker-stop` - Stop and remove the running container
- `make docker-compose-up` - Start production services with Docker Compose
- `make docker-compose-down` - Stop Docker Compose services
- `make docker-compose-dev` - Start development environment with live reload
- `make docker-clean` - Remove Docker images (preserves data volumes)
- `make docker-clean-all` - Remove all Docker images and volumes (destructive)

**Quick Start with Docker:**
```bash
cp config.docker.example.json config.docker.json
docker-compose up
```

**Development with live reload:**
```bash
docker-compose -f docker-compose.dev.yml up
```

**Docker Files:**
- `Dockerfile` - Multi-stage production build (Go 1.24 + Alpine)
- `Dockerfile.dev` - Development image with Air live reload
- `docker-compose.yml` - Production configuration with volumes and health check
- `docker-compose.dev.yml` - Development setup with source mounting
- `.dockerignore` - Files excluded from Docker context
- `.air.docker.toml` - Air live reload configuration for Docker development

## Debugging

Use Delve for debugging:

```bash
# Install Delve
go install github.com/go-delve/delve/cmd/dlv@latest

# Build the app
make build

# Start the debugger
dlv --listen=:2345 --headless=true --api-version=2 --accept-multiclient exec ./bin/maglev
```

Then connect from GoLand IDE or other Delve-compatible debugger.

## Important: Requirements to make a commit

Before committing any code, you must always run all of these steps, and have them all succeed:

1. Run `make lint` and fix any linting issues that are identified
2. Run `make test` and fix any failing tests
3. Run `go fmt ./...` and commit all of the formatting changes

## Architecture Overview

This is a Go 1.24.2+ application that provides a REST API for OneBusAway transit data. The architecture follows a layered design:

### File Structure

```
maglev/
├── cmd/api/              # Application entry point
├── internal/
│   ├── app/              # Application container (dependency injection)
│   ├── appconf/          # Configuration management
│   ├── gtfs/             # GTFS data management (static + real-time)
│   ├── logging/          # Structured logging and error handling
│   ├── models/           # Business models and API response structures
│   ├── restapi/          # HTTP handlers and middleware
│   ├── utils/            # Helper functions (geometry, ID parsing, validation)
│   └── webui/            # Web interface handlers
├── gtfsdb/               # SQLite database layer (sqlc-generated)
└── testdata/             # Test fixtures (RABA GTFS data, protobuf files)
```

### Core Components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OneBusAway/maglev](https://github.com/OneBusAway/maglev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
