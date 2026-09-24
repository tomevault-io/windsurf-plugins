---
trigger: always_on
description: provides multi-tenant, API-driven bare-metal lifecycle management, working in
---

# AGENTS.md

This file provides guidance for AI coding agents working in the
`rest-api/` tree of the `infra-controller` repository.

## Project Overview

**NVIDIA Infrastructure Controller REST** is a collection of Go microservices that comprise
the management backend for NVIDIA Infrastructure Controller (NICo), exposed as a REST API. It
provides multi-tenant, API-driven bare-metal lifecycle management, working in
concert with Core services for on-site hardware operations.

> **Status:** Active development. APIs, configurations, and features may
> change without notice between releases.

### Key Responsibilities

- REST API for hardware inventory, provisioning, and lifecycle orchestration
- Multi-tenant site and instance management
- Temporal-based cloud and site workflow orchestration
- On-site agent for datacenter-local operations
- IP address management (IPAM)
- Authentication and authorization (Keycloak, JWT, service accounts)
- Native PKI certificate management
- CLI client (`nicocli`) with interactive TUI

## Repository Structure

```text
rest-api/
├── api/                  # Main REST API server (Echo-based)
├── auth/                 # Authentication (Keycloak, JWT, service accounts)
├── cert-manager/         # Native PKI certificate management (credsmgr)
├── cli/                  # CLI client (nicocli) with TUI
├── common/               # Shared utilities and configuration
├── db/                   # Database layer (Bun ORM, pgx, migrations)
├── deploy/               # Kubernetes deployment (Kind, Kustomize, Helm)
├── docker/               # Dockerfiles (local dev and production)
├── helm/                 # Helm charts for Kubernetes deployment
├── ipam/                 # IP address management
├── nvswitch-manager/     # NVSwitch firmware management (NSM)
├── openapi/              # OpenAPI spec and SDK generation
├── powershelf-manager/   # Power shelf management (PSM)
├── flow/                 # Carbide Flow logic
├── sdk/                  # Go API client (simple and standard variants)
├── site-agent/           # On-site agent for datacenter
├── site-manager/         # Site management service (sitemgr)
├── site-workflow/        # Site-level Temporal workflows
├── temporal-helm/        # Temporal Helm chart
├── workflow/             # Cloud Temporal workflows and activities
├── workflow-schema/      # Protobuf and workflow schemas
├── .github/              # GitHub Actions workflows and templates
├── Makefile              # Primary build/task automation
└── go.mod                # Go module and dependency management
```

## Technology Stack

- **Language:** Go (version specified in `go.mod`; module `github.com/NVIDIA/infra-controller/rest-api`)
- **HTTP framework:** Echo v4 (with middleware for CORS, auth, rate limiting, audit)
- **Database:** PostgreSQL via pgx v5 (connection pool) and Bun ORM (queries, migrations)
- **Workflow engine:** Temporal (cloud and site workflows/activities)
- **gRPC:** Connect-RPC and google.golang.org/grpc (site-agent, workflow schemas)
- **Protobuf:** buf for code generation
- **Observability:** OpenTelemetry, Prometheus (echoprometheus), Sentry
- **Auth:** Keycloak, JWT
- **Testing:** testify (assert/require/suite), go-sqlmock, testcontainers-go, gomock
- **Build tool:** Make

## Build, Test, and Lint Commands

### Building

```bash
# Build all binaries (linux/amd64, static)
make build

# Build and install CLI to $GOPATH/bin
make nico-cli

# Build Docker images (production)
make docker-build

# Build Docker images (local dev, public base images)
make docker-build-local
```

### Testing

```bash
# Run all tests (auto-manages PostgreSQL container)
make test

# Module-level tests
make test-api
make test-db
make test-workflow
make test-auth
make test-common
make test-cert-manager
make test-site-agent        # starts mock Core and Flow gRPC servers first
make test-site-manager
make test-site-workflow
make test-ipam

# PostgreSQL management for tests
make postgres-up            # start test PostgreSQL container
make postgres-down          # stop test PostgreSQL container
make ensure-postgres        # start if not running, wait until ready
make migrate                # run database migrations against test DB
```

Tests require a PostgreSQL container (postgres:14.4-alpine) on port 30432.
The Makefile manages this automatically via `ensure-postgres`.

Use these targets rather than calling `go test` yourself, because they start what the tests
need and skipping that setup does not fail fast:

- `make test-site-agent` starts mock Core and Flow gRPC servers. Without them the
  `site-agent/pkg/components` tests retry the connection on a `40s` backoff until the `10m`
  test timeout, so a bare `go test ./site-agent/...` looks like a hang rather than an error.
  That target also scopes to `site-agent/pkg/components` and sets `CGO_ENABLED=1` for `-race`,
  so it is not the same package set or the same build.
- `test-api`, `test-auth`, `test-db`, `test-flow`, `test-ipam`, `test-nvswitch-manager`,
  `test-powershelf-manager`, and `test-workflow` call `ensure-postgres` first.
- Every Postgres-backed package resets the schema, so packages running in parallel drop and
  recreate the same tables and fail in `TestSetupSchema`. Pass `-p 1` whenever you do run

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dsx-ai-factory/infra-controller](https://github.com/dsx-ai-factory/infra-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
