---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

### Running the Service

```bash
make start-service                # Start service on default port 8080
PORT=3000 make start-service      # Start service on custom port
make stop-service                 # Stop service
go run cmd/eval_hub/main.go       # Direct Go execution
```

### Building

```bash
make build              # Build all binaries (service, init, sidecar, mcp) into bin/
make build-service      # Build only the API service binary
make build-mcp          # Build only the evalhub-mcp MCP server binary
./bin/eval-hub          # Run the API service binary
./bin/evalhub-mcp       # Run the MCP server binary
```

### Testing

```bash
make test               # Run unit tests (./auth/..., ./internal/..., ./cmd/...)
make test-fvt           # Run FVT tests using godog (tests/features/...)
make test-all           # Run unit tests, FVT, then FVT against a started server (test-fvt-server)
make test-coverage      # HTML reports: bin/coverage.html and bin/coverage-init.html

# Run specific unit test
go test -v ./internal/eval_hub/handlers -run TestHandleName

# Run specific FVT test
go test -v ./tests/features -run TestFeatureName
```

### Code Quality

```bash
make fmt                # Format code with go fmt
make lint               # Run go vet
make vet                # Run go vet (same as lint)
```

**Always run `make fmt lint` after file changes and before committing.** This ensures consistent formatting and catches issues early.

### Go Version

**Do not modify the Go version in `go.mod`.** The version specified there is the source of truth. If your local Go toolchain is older, use `GOTOOLCHAIN=auto` to let Go automatically download the required version. Never downgrade `go.mod` to match a locally installed toolchain.

### Dependencies

```bash
make install-deps       # Download and tidy dependencies (requires Python 3 for test color output via scripts/grcat)
make update-deps        # Update all dependencies to latest
# Note: uv (https://docs.astral.sh/uv/) is required for `make test-fvt` and `make start-service` (manages Python venv and test dependencies)
```

### Database Setup

Directory: `tests/postgres` (run these targets from that directory, e.g. `cd tests/postgres`).

```bash
make install-postgres   # Install PostgreSQL (macOS/Linux)
make start-postgres     # Start PostgreSQL service
make stop-postgres      # Stop PostgreSQL service
make create-database    # Create eval_hub database
make create-user        # Create eval_hub user
make grant-permissions  # Grant permissions to user
```

### Cleanup

```bash
make clean              # Remove build artifacts and coverage files
```

## Git commits

Use [Conventional Commits](https://www.conventionalcommits.org/) with an optional scope (e.g. `feat(http): …`). Accepted type prefixes: `build`, `bump`, `chore`, `ci`, `docs`, `feat`, `fix`, `perf`, `refactor`, `revert`, `style`, `test`.

When a change is assisted by Cursor, add one of these lines, as appropriate, to the **end** of the commit message body (after the subject and any description), as Git trailers:

```text
Assisted-by: Cursor
Made-with: Cursor
Generated with: Claude Code
```

## Architecture Overview

### Project Structure

This project follows the standard Go project layout with a clear separation between public entry points (`cmd/`) and private application code (`internal/`). See **ARCHITECTURE.md** for a concise layout and request flow.

- **cmd/eval_hub/** - Main API service entry point
- **cmd/evalhub_mcp/** - MCP server entry point (stdio and HTTP transports)
- **cmd/eval_runtime_init/** - Init container for Kubernetes job pods
- **cmd/eval_runtime_sidecar/** - Sidecar for job pods (proxy, readiness, termination log)
- **pkg/api/** - Shared API types (IDs, errors, request/response shapes)
- **pkg/evalhubclient/** - HTTP client library for the eval-hub REST API (used by MCP server and external consumers)
- **auth/** - Authentication configuration and HTTP middleware helpers
- **internal/eval_hub/abstractions/** - `Storage`, `Runtime`, and related interfaces
- **internal/eval_hub/config/** - Configuration loading with Viper
- **internal/eval_hub/constants/** - Shared constants (log field names, etc.)
- **internal/eval_hub/executioncontext/** - Per-request execution context (`Ctx`, logger, `User`, `Tenant`, etc.)
- **internal/eval_hub/handlers/** - HTTP handlers (depend on `Handlers` for config, storage, runtime)
- **internal/eval_hub/http_wrappers/** - `RequestWrapper` / `ResponseWrapper` abstractions for handlers
- **internal/eval_hub/runtimes/** - Local and Kubernetes runtime implementations
- **internal/eval_hub/storage/** - Persistence implementations (e.g. SQL)
- **internal/logging/** - Logger creation (zap backend, `slog` API)
- **internal/eval_hub/metrics/** - Prometheus metrics and middleware
- **internal/eval_hub/server/** - Server setup, routing, auth wiring, `newExecutionContext`
- **internal/evalhub_mcp/config/** - MCP server configuration (CLI flags, YAML profiles, env vars)
- **internal/evalhub_mcp/server/** - MCP server setup (transport selection, capabilities, client wiring)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eval-hub/eval-hub](https://github.com/eval-hub/eval-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
