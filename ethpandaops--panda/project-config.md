---
trigger: always_on
description: This file provides guidance to coding agents working in this repository.
---

# Repository Guide

This file provides guidance to coding agents working in this repository.

## Project Overview

ethpandaops/panda is a server + proxy system for Ethereum analytics. The server is the only product API boundary, runs sandboxed Python locally, and delegates credentialed upstream access to a separate proxy.

The architecture is:
- `panda` talks to `server`
- `server` routes through `proxy`/`proxies`
- each `proxy` talks to the datasources it advertises

Modules provide integration-specific metadata and behavior for ClickHouse, Prometheus, Loki, Dora, Forky, Ethnode, CBT, and block archive.

See `docs/architecture.md` for the canonical boundary definition.

## Architectural Guardrails

- `server` is the only public/runtime API boundary for `panda`, MCP clients, and sandbox code
- sandboxed Python calls back into `server`, never directly into `proxy`
- `proxy` is a thin credentialed upstream gateway, not a product operations API
- module behavior is exposed through `execute_python`, resources, docs, and search; do not add per-module MCP tools
- datasource identity is owned by the proxy route that advertised it; modules initialize from proxy discovery
- rendered content never branches on caller identity: resource handlers receive a `surface.Dialect` and spell invocations through its vocabulary (`pkg/surface`); never add `if cli/mcp` conditionals to content code

## Supported Deployment Modes

Only two deployment modes are supported:

1. all local: `panda -> local server -> local proxy`
2. local server + hosted proxy: `panda -> local server -> hosted proxy`

In both modes, sandbox code still executes locally and calls back into local `server`.

## Commands

```bash
# Build
make build                    # Build panda-server + panda
make build-proxy             # Build standalone proxy binary
make docker                   # Build Docker image
make docker-sandbox           # Build sandbox container image
make install                  # Install panda-server + panda binaries into GOBIN

# Test
make test                     # Run tests with race detector
make test-coverage            # Run tests with coverage report
go test -race -v ./pkg/sandbox/...  # Run tests for a single package

# Lint and format
make lint                     # Run golangci-lint (v2 config)
make lint-fix                 # Run golangci-lint with auto-fix
make fmt                      # Format code (gofmt -s)
make vet                      # Run go vet

# Run
make run                      # Build + run server
docker compose up -d          # Full local stack: server + proxy

# CLI (requires a running server)
./panda datasources                          # List available datasources
./panda schema                               # Show ClickHouse table schemas
./panda docs                                 # Show Python API docs
./panda execute --code 'print("hello")'      # Execute Python in sandbox
./panda session list                         # Manage sandbox sessions
./panda search examples "block count"        # Semantic search examples
./panda search runbooks "finality delay"     # Semantic search runbooks

# Evaluation tests (in tests/eval/)
cd tests/eval && uv sync
uv run python -m scripts.eval --tags smoke                # single-pass eval (CI smoke uses this)
uv run python -m scripts.eval                             # full eval over every case in cases/*.yaml
uv run python -m scripts.harden                           # same harness, wrapped in the optimization loop
uv run python -m scripts.repl
```

## Architecture

### Key Components

- **App kernel** (`pkg/app/`): Shared server-side initialization for the module registry, proxy client, sandbox, cartographoor, and search indices
- **Server** (`pkg/server/`, `cmd/server/`): Control plane for MCP (SSE + streamable-http), product HTTP API, resource/tool registration, and sandbox orchestration
- **CLI** (`pkg/cli/`, `cmd/panda/`): HTTP client for the server API with human-friendly output
- **Credential proxy** (`pkg/proxy/`, `cmd/proxy/`): Trust boundary that holds datasource credentials and executes raw upstream requests on behalf of the server
- **Storage** (`pkg/storage/`): Local file storage for sandbox outputs, backed by afero filesystem
- **Sandbox** (`pkg/sandbox/`): Data plane that executes Python in isolated containers (Docker for dev, gVisor for production, `none` to disable execution)
- **Modules** (`modules/`): Per-integration packages that provide config, examples, docs, resources, and server-side operation behavior

### Data Flow

1. `panda` or an MCP client connects to `server`
2. `server` builds a credential-free sandbox environment with server runtime tokens and datasource metadata
3. sandbox code calls back into `server` for operations and storage
4. `server` stores uploaded files locally via the storage service (`~/.panda/data/storage/`)
5. `server` routes through the proxy client/router for credentialed upstream access to ClickHouse, Prometheus, Loki, Ethnode, and Benchmarkoor

### Module System

Twelve compiled-in modules are registered in `pkg/app/app.go`:
- `clickhouse`
- `prometheus`
- `loki`
- `dora`
- `forky`
- `ethnode`
- `cbt`
- `benchmarkoor`
- `tracoor`
- `block_archive`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ethpandaops/panda](https://github.com/ethpandaops/panda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
