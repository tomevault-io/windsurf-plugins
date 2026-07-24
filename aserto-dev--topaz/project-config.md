---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Topaz is an open-source authorization service providing fine-grained, real-time, policy-based access control for applications and APIs. It uses Open Policy Agent (OPA) as its decision engine and provides a built-in directory inspired by Google Zanzibar's data model.

**Core Technologies:**
- Go 1.24+ (toolchain 1.25+)
- OPA (Open Policy Agent) for policy evaluation
- BoltDB (bbolt) for embedded directory storage
- gRPC and gRPC-Gateway for APIs
- Docker for containerized deployment
- Google Wire for dependency injection

## Build and Development Commands

### Building
```bash
# Build for current platform (requires Go 1.25+ in environment)
make build

# Build output: dist/build_<os>_<arch>/topaz
./dist/build_linux_amd64/topaz --help
```

### Testing
```bash
# Run all tests (builds test snapshot, runs integration tests)
make test

# Run specific test suites
make run-tests

# Run only unit tests without building snapshot
./.ext/bin/gotestsum --format short-verbose -- -count=1 -timeout 120s -parallel=1 -v ./pkg/app/tests/...

# Test single package
go test -v ./pkg/app/tests/ds -count=1
```

### Linting
```bash
# Run golangci-lint
make lint
```

### Dependencies
```bash
# Install all development dependencies (vault, svu, goreleaser, golangci-lint, gotestsum, wire, etc.)
make deps

# Tidy go.mod files
make go-mod-tidy
```

### Code Generation
```bash
# Generate wire dependency injection code
make generate

# Wire generates code in:
# - pkg/app/topaz/wire_gen.go
# - pkg/cc/wire_gen.go
```

### Container Operations
```bash
# Build test snapshot container
make test-snapshot
# Output: ghcr.io/aserto-dev/topaz:0.0.0-test-<git-sha>-<arch>

# Run local test snapshot
make run-test-snapshot
make start-test-snapshot
```

## Architecture

### High-Level Structure

Topaz consists of four main executables:

1. **topaz** (`cmd/topaz/main.go`) - CLI for managing Topaz instances
2. **topazd** (`cmd/topazd/main.go`) - Topaz daemon server
3. **topaz-db** (`cmd/topaz-db/`) - Database utilities
4. **topaz-backup** (`cmd/topaz-backup/`) - Backup utilities

### Core Packages

- **pkg/app/** - Application layer
  - `authorizer.go` - Authorization service implementation
  - `edgedir.go` - Edge directory service (local Zanzibar-style directory)
  - `console.go` - Web console service
  - `impl/` - Authorizer endpoint implementations (is, query, compile, decisiontree)
  - `tests/` - Integration tests for all services

- **pkg/cli/** - CLI command implementations
  - `cmd/` - All CLI commands (run, start, stop, templates, directory, authorizer, etc.)
  - `cc/` - Common CLI context and configuration
  - `x/` - CLI utilities and constants

- **pkg/cc/** - Core configuration and context
  - `config/` - Configuration schema and parsing
  - Wire-injected context management

- **pkg/service/builder/** - Service factory and manager for GRPC/HTTP services

- **builtins/** - Custom OPA built-in functions
  - `edge/` - Edge directory built-ins (ds.object, ds.relation, ds.check, ds.user, etc.)
  - `helper.go` - Utilities for OPA integration

- **resolvers/** - Resolvers for runtime and directory dependencies

- **plugins/** - OPA plugins
  - `edge/` - Edge directory plugin
  - `decisionlog/` - Decision logging plugin
  - `noop/` - No-op implementations

- **controller/** - Controller for managing service lifecycle

- **directory/** - Directory identity resolution

- **decisionlog/** - Decision logging infrastructure

### Dependency Injection with Wire

The project uses Google Wire for compile-time dependency injection:

- Wire definitions: `pkg/app/topaz/wire.go`, `pkg/cc/wire.go`
- Generated code: `pkg/app/topaz/wire_gen.go`, `pkg/cc/wire_gen.go`
- When modifying dependencies, run `make generate` to regenerate wire code

### Service Architecture

Topaz exposes multiple gRPC services with HTTP/REST gateways:

1. **Authorizer Service** (default: grpc:8282, http:8383)
   - Authorization decisions (is, query, compile, decisiontree)
   - Policy management
   - AuthZ API v2

2. **Directory Service** (default: grpc:9292, http:9393)
   - Reader, Writer, Importer, Exporter, Model APIs
   - Directory v3 API
   - AuthZen Access API

3. **Console Service** (default: http:8080)
   - Web UI for managing directory and viewing policies

4. **Health Service** (default: http:9494)

5. **Metrics Service** (default: http:9696)
   - Prometheus metrics

### Configuration

Configuration files use YAML format (version 2 schema):
- Schema: `pkg/cc/config/schema/config.yaml`
- Examples: `docs/examples/config-*.yaml`
- Environment variables: All config values can be set with `TOPAZ_*` prefix
- Default paths:
  - `$HOME/.config/topaz/` - Config directory
  - `$HOME/.config/topaz/certs/` - TLS certificates
  - `$HOME/.config/topaz/db/` - Database files

Key configuration sections:
- `logging` - Log levels (prod, log_level, grpc_log_level)
- `directory` - Edge directory settings (db_path, request_timeout)
- `remote_directory` - Remote directory connection (for identity resolution)
- `api.services` - Service endpoints (authorizer, reader, writer, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aserto-dev/topaz](https://github.com/aserto-dev/topaz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
