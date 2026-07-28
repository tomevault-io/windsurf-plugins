---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

E2B Infrastructure is the backend infrastructure powering E2B (e2b.dev), an open-source cloud platform for AI code interpreting. It provides sandboxed execution environments using Firecracker microVMs, deployed on GCP using Terraform and Nomad.

**Start with [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)** — it explains what each service does, how services interact (with diagrams of the core flows: sandbox creation, traffic routing, pause/resume, template builds), and the deployment topology. Reading it first is the fastest way to understand this repository.

**Keep docs/ARCHITECTURE.md updated**: if your change alters anything it describes (service responsibilities, ports, protocols, data stores, flows, deployment topology), update the document in the same change.

## Common Development Commands

### Setup & Environment
```bash
# Switch between environments (prod, staging, dev)
make switch-env ENV=staging

# Setup GCP authentication
make login-gcloud

# Initialize Terraform
make init

# Setup local development stack (PostgreSQL, Redis, ClickHouse, monitoring)
make local-infra
```

### Building & Testing
```bash
# Run all unit tests across packages
make test

# Run integration tests
make test-integration

# Build specific package
make build/api
make build/orchestrator

# Generate code (proto, SQL, OpenAPI)
make generate

# Format and lint code
make fmt
make lint

# Regenerate mocks
make generate-mocks

# Tidy go dependencies
make tidy
```

### Running Services Locally
```bash
# From packages/api/
make run-local          # Run API server on :3000
make dev                # Run with air (hot reload)

# From packages/orchestrator/
make run-local          # Run orchestrator
make run-debug          # Run with race detector
```

### Package-Specific Commands
```bash
# API: Generate OpenAPI code
cd packages/api && make generate

# Orchestrator: Generate proto + OpenAPI
cd packages/orchestrator && make generate

# DB: Run migrations
make migrate

# Run single test
cd packages/<package> && go test -v -run TestName ./path/to/package
```

### Deployment
```bash
# Build and upload all services to GCP
make build-and-upload

# Build specific service
make build-and-upload/api
make build-and-upload/orchestrator

# Plan Terraform changes
make plan                    # All changes
make plan-without-jobs       # Without Nomad jobs
make plan-only-jobs          # Only Nomad jobs

# Apply changes
make apply
```

## Architecture Overview

### Service Communication Flow
```
Client → Client-Proxy → API (REST) ⟷ PostgreSQL
                      ↓              ⟷ Redis
                   Orchestrator     ⟷ ClickHouse
                      ↓ (gRPC)
                   Firecracker VMs
                      ↓
                   Envd (in-VM daemon)
```

### Core Services

**API (`packages/api/`)** - REST API using Gin framework
- Entry point: `main.go`
- Core logic: `internal/handlers/store.go` (APIStore)
- Authentication: API keys, access tokens, and OIDC auth provider JWTs
- OpenAPI code generation: `internal/api/*.gen.go`
- Port: 80

**Orchestrator (`packages/orchestrator/`)** - Firecracker microVM orchestration
- Entry point: `main.go`
- VM management: `pkg/sandbox/`
- Firecracker integration: `pkg/sandbox/fc/`
- Networking: `pkg/sandbox/network/`
- Storage: `pkg/sandbox/nbd/` (Network Block Device)
- Template caching: `pkg/sandbox/template/`
- Template building: `pkg/template/`
- gRPC server: `pkg/server/`
- Utilities: `cmd/clean-nfs-cache/`, `cmd/inspect-build/`, `cmd/dummy-orchestrator/`

**Envd (`packages/envd/`)** - In-VM daemon using Connect RPC
- Runs inside each Firecracker VM
- Process management API: `packages/envd/spec/process/process.proto`
- Filesystem API: `packages/envd/spec/filesystem/filesystem.proto`
- Port: 49983
- **Version in `pkg/version.go` must be bumped on every behavioral change** (not comments/docs-only changes)

**Client Proxy (`packages/client-proxy/`)** - Edge routing layer
- Service discovery via Consul
- Request routing to orchestrators
- Redis-backed state management

**Shared (`packages/shared/`)** - Common utilities
- Proto definitions: `pkg/grpc/orchestrator/`, `pkg/grpc/envd/`
- Telemetry: `pkg/telemetry/` (OpenTelemetry)
- Logging: `pkg/logger/` (Zap + OTEL)
- Database: `pkg/db/` (ent ORM)
- Models: `pkg/models/`
- Storage: `pkg/storage/` (GCS/S3 clients)
- Feature flags: `pkg/featureflags/` (LaunchDarkly)

**Database (`packages/db/`)** - PostgreSQL layer
- Migrations: `migrations/*.sql` (goose)
- Queries: `queries/*.sql` (sqlc)
- Generated code: `queries/` (plus `pkg/auth/queries/`, `pkg/dashboard/queries/`)

### Key Technologies

- **go 1.26.5** with workspaces (`go.work`)
- **Firecracker** for microVM virtualization
- **PostgreSQL** for primary data (sqlc for queries)
- **ClickHouse** for analytics
- **Redis** for caching and state
- **Terraform + Nomad** for IaC and orchestration
- **OpenTelemetry** for observability (Grafana stack: Loki, Tempo, Mimir)
- **gRPC/Connect RPC** for service communication
- **Gin** (API), **chi** (Envd) for HTTP

### Code Generation

The codebase uses several code generators:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [e2b-dev/infra](https://github.com/e2b-dev/infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
