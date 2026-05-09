---
trigger: always_on
description: provides multi-tenant, API-driven bare-metal lifecycle management, working in
---

# AGENTS.md

This file provides guidance for AI coding agents working in the
`infra-controller-rest` repository.

## Project Overview

**NVIDIA Infra Controller REST** is a collection of Go microservices that comprise
the management backend for NVIDIA Infra Controller, exposed as a REST API. It
provides multi-tenant, API-driven bare-metal lifecycle management, working in
concert with [NVIDIA Infra Controller Core](https://github.com/NVIDIA/infra-controller-core)
for on-site hardware operations.

> **Status:** Experimental/Preview. APIs, configurations, and features may
> change without notice between releases.

### Key Responsibilities

- REST API for hardware inventory, provisioning, and lifecycle orchestration
- Multi-tenant site and instance management
- Temporal-based cloud and site workflow orchestration
- On-site agent for datacenter-local operations
- IP address management (IPAM)
- Authentication and authorization (Keycloak, JWT, service accounts)
- Native PKI certificate management
- CLI client (`carbidecli`) with interactive TUI

## Repository Structure

```text
infra-controller-rest/
├── api/                  # Main REST API server (Echo-based)
├── auth/                 # Authentication (Keycloak, JWT, service accounts)
├── cert-manager/         # Native PKI certificate management (credsmgr)
├── cli/                  # CLI client (carbidecli) with TUI
├── common/               # Shared utilities and configuration
├── db/                   # Database layer (Bun ORM, pgx, migrations)
├── deploy/               # Kubernetes deployment (Kind, Kustomize, Helm)
├── docker/               # Dockerfiles (local dev and production)
├── helm/                 # Helm charts for Kubernetes deployment
├── ipam/                 # IP address management
├── nvswitch-manager/     # NVSwitch firmware management (NSM)
├── openapi/              # OpenAPI spec and SDK generation
├── powershelf-manager/   # Power shelf management (PSM)
├── rla/                  # Rack Level Agent (RLA) logic
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

- **Language:** Go (version specified in `go.mod`; module `github.com/NVIDIA/ncx-infra-controller-rest`)
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
make carbide-cli

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
make test-site-agent        # requires mock gRPC servers
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

### Linting and Formatting

```bash
# Check formatting (fails if repo is dirty after go fmt)
make fmt-go

# Run all linters (go vet + golangci-lint + revive)
make lint-go

# Auto-fix formatting
go fmt ./...
```

### OpenAPI

```bash
# Lint the OpenAPI spec
make lint-openapi

# Preview in Redoc UI (http://127.0.0.1:8090)
make preview-openapi

# Generate Go SDK from OpenAPI spec
make generate-sdk

# Publish OpenAPI docs
make publish-openapi
```

### Protobuf Code Generation

```bash
make carbide-proto          # fetch proto files from carbide-core
make carbide-protogen       # generate Go code from protos
make rla-proto              # fetch RLA proto files
make rla-protogen           # generate Go code from RLA protos
```

### Local Development (Kind cluster)

```bash
make kind-reset             # full reset: cluster + infra + Helm deploy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/infra-controller-rest](https://github.com/NVIDIA/infra-controller-rest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
