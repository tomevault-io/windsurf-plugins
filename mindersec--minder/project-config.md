---
trigger: always_on
description: Minder is an open-source supply chain security platform that helps development teams build more secure software and prove their security posture. It enables proactive security policy management across repositories and artifacts, with features for continuous security enforcement, artifact attestation, and dependency management.
---

# Minder Development Guide for Claude Code

## Project Overview

Minder is an open-source supply chain security platform that helps development teams build more secure software and prove their security posture. It enables proactive security policy management across repositories and artifacts, with features for continuous security enforcement, artifact attestation, and dependency management.

**Key Technologies:**
- **Language**: Go 1.24+
- **Protocol Buffers**: gRPC for API communication, REST via grpc-gateway
- **Database**: PostgreSQL with sqlc for type-safe SQL
- **Message Queue**: NATS JetStream for event-driven architecture
- **Authentication**: Keycloak (OAuth2/OIDC), JWT tokens
- **Authorization**: OpenFGA (relationship-based access control)
- **Frontend CLI**: Cobra framework with Bubble Tea TUI components
- **Observability**: OpenTelemetry, Prometheus metrics, zerolog
- **Security**: Sigstore for artifact signing/verification

## Architecture

Minder consists of:

1. **minder-server**: gRPC/REST API server (control plane)
2. **minder CLI**: Command-line interface for users
3. **reminder**: Background service for scheduled tasks
4. **Control Plane** (`internal/controlplane/`): API handlers and business logic
5. **Engine** (`internal/engine/`): Policy evaluation and enforcement
6. **Providers** (`internal/providers/`): Integration with GitHub, GitLab, container registries
7. **Datasources** (`internal/datasources/`): REST API data fetching and ingestion

## Directory Structure

```
.
├── cmd/                    # Main applications
│   ├── cli/               # minder CLI tool
│   ├── server/            # minder-server (gRPC/REST API)
│   ├── dev/               # Development utilities
│   └── reminder/          # Scheduled task service
├── internal/              # Private application code
│   ├── controlplane/      # API handlers and orchestration
│   ├── engine/            # Policy evaluation engine
│   ├── providers/         # GitHub, GitLab, container registry integrations
│   ├── datasources/       # REST API datasources
│   ├── db/                # Generated SQLC database code
│   ├── auth/              # Authentication (Keycloak, JWT)
│   ├── authz/             # Authorization (OpenFGA)
│   ├── events/            # Event handling (NATS)
│   ├── entities/          # Entity management
│   ├── reconcilers/       # State reconciliation
│   └── crypto/            # Cryptographic operations
├── pkg/                   # Public library code
│   ├── api/               # Generated protobuf/OpenAPI code
│   ├── profiles/          # Security profiles
│   ├── ruletypes/         # Rule type definitions
│   ├── mindpak/           # Package management
│   ├── engine/            # Engine interfaces
│   └── providers/         # Provider interfaces
├── proto/                 # Protocol buffer definitions
├── database/              # Database layer
│   ├── migrations/        # SQL migrations (golang-migrate)
│   ├── query/             # SQLC query definitions
│   └── schema/            # Database schema
├── deployment/            # Kubernetes and Helm charts
├── docs/                  # Documentation (Docusaurus)
├── examples/              # Example configurations
└── .mk/                   # Makefile includes
```

## Development Workflow

### Prerequisites

**Required tools:**
- Go 1.24+
- Docker & Docker Compose
- OpenSSL (for key generation)

**Build tools (installed via `make bootstrap`):**
- ko (for container builds)
- buf (Protocol Buffer compilation)
- sqlc (SQL code generation)
- golangci-lint (linting)
- gotestfmt (test output formatting)
- protoc plugins (grpc-gateway, protoc-gen-go, etc.)
- mockgen (mock generation)
- yq (YAML processing)
- fga (OpenFGA CLI)
- helm-docs (Helm documentation)

**Runtime services (via Docker):**
- PostgreSQL (database)
- Keycloak (authentication)
- NATS (message queue)

### Initial Setup

Before building or running Minder, install all build dependencies and initialize configuration:

```bash
# Install build tools and initialize configuration
make bootstrap
```

This command will:
- Install all Go-based build tools (sqlc, protoc plugins, mockgen, etc.)
- Create `config.yaml` and `server-config.yaml` from example templates (if they don't exist)
- Generate encryption keys in `.ssh/` directory for token signing

**Note**: Run `make bootstrap` once after cloning the repository. You may need to run it again if build tool versions change.

### Building

```bash
# Build both CLI and server binaries to ./bin/
make build

# Clean build artifacts
make clean
```

### Code Generation

Code generation is critical and must be run after changes to:

```bash
# Run all code generation (protobuf, sqlc, mocks, OpenAPI)
make gen

# Individual generators:
make buf        # Generate protobuf code from proto/
make sqlc       # Generate Go code from database/query/*.sql
make mock       # Generate mocks using mockgen
make oapi       # Generate OpenAPI client code
make cli-docs   # Generate CLI documentation
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mindersec/minder](https://github.com/mindersec/minder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
