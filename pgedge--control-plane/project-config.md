---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The pgEdge Control Plane is a distributed system for managing and orchestrating Postgres databases. It uses Docker Swarm to deploy databases as Docker containers, however it is architected to support other orchestrators in the future. It uses an embedded Etcd server for distributed state management, and provides declarative APIs for database lifecycle management. The system supports multi-active deployments with Spock replication, read replicas via Patroni, and backup/restore operations through pgBackRest.

## Common Development Commands

### Local Development
```bash
# Start local Docker compose-based development environment in foreground with hot reload
make dev-watch

# Rebuild and restart after changes
make dev-build

# Stop development environment
make dev-down

# Reset development environment (removes databases, networks, data)
make dev-reset
```

### Testing
```bash
# Run unit tests
make test

# Run unit tests with specific package
go test ./server/internal/database/...

# Run etcd lifecycle tests
make test-etcd-lifecycle

# Run workflow backend tests
make test-workflows-backend

# Deploy the Lima-based E2E test fixture
make deploy-lima-fixture

# Update the Control Plane on an existing Lima-based test fixture
make update-lima-fixture

# Run E2E tests against the Lima-based test fixture
make test-e2e E2E_FIXTURE=lima

# Run specific E2E test
make test-e2e E2E_FIXTURE=lima E2E_RUN=TestCancelDatabaseTask

# Run E2E tests with debug output for failing tests and skip cleanup
make test-e2e E2E_DEBUG=1

# Run cluster integration tests
make test-cluster

# Run specific cluster test
make test-cluster CLUSTER_TEST_RUN=TestInitialization
```

### Linting and Code Quality
```bash
# Run linters
make lint

# Update license notices
make licenses
```

### API Code Generation
```bash
# Regenerate API code from Goa design files
make -C api generate

# This generates:
# - Service interfaces and endpoints in api/apiv1/gen/control_plane/
# - HTTP transport layer in api/apiv1/gen/http/
# - OpenAPI 3.0 specifications (JSON and YAML)
```

### Documentation

```bash
# Build and serve documentation locally at http://localhost:8000
make docs
```

## Git Commit Message Style

- Commit message headers should follow the Conventional Commit style, e.g. `feat: an awesome feature`.
- Try to keep commit message headers to 50 characters or less.
- Commit message bodies should be markdown-formatted.
- Wrap each commit body line at 72 characters unless it would be syntactically incorrect, such as a hyperlink or code example.
- Ticket numbers should be included in the commit message footer, e.g. `PLAT-123`.

## Branch Naming Style

Branch names should include a Conventional Commit type, a ticket number (if any), and a brief, lower kebab-cased description, e.g. `feat/PLAT-123/an-awesome-feature` or `feat/an-awesome-feature` if there is no ticket.

## Architecture Overview

### Hexagonal Architecture

The codebase follows hexagonal/ports & adapters architecture with clear separation between:
- **Domain logic**: Core business logic in `server/internal/`
- **Infrastructure**: Orchestrator implementations, storage backends
- **API layer**: Goa-generated HTTP/MQTT interfaces

### Key Architectural Patterns

**Dependency Injection**: Uses `samber/do` injector. Each package has a `Provide()` function that registers dependencies with the injector.

**Resource Lifecycle**: Resources follow a standard lifecycle:
1. **Refresh**: Sync current state from infrastructure
2. **Plan**: Compute diff between desired and current state
3. **Apply**: Execute Create/Update/Delete operations
4. Resources declare their Executor (where they run), Dependencies, and lifecycle methods

**Workflow Orchestration**: Built on `cschleiden/go-workflows`:
- Workflows represent long-running operations (database creation, updates, backups)
- Activities are atomic units of work that execute on specific hosts
- Workflows persist state to etcd for durability and resumability
- Task tracking provides visibility into workflow execution progress

**State Management**:
- etcd stores all cluster state with versioned values and watch support
- Storage layer provides transaction support and optimistic locking
- Declarative desired state reconciliation pattern

### Directory Structure

**server/internal/** - Core server implementation:
- **api/** - API server, mounts Goa-generated handlers (HTTP & MQTT)
- **app/** - Application lifecycle, orchestrates startup (pre-init vs post-init states)
- **config/** - Multi-source configuration (JSON, env vars, CLI flags)
- **etcd/** - Embedded and remote etcd client, cluster membership, RBAC
- **storage/** - Generic storage interface abstracting etcd (Get, Put, Delete, Watch, Txn)
- **database/** - Database domain models, specs, instance state management
- **task/** - Task tracking system for workflow visibility and audit logs
- **orchestrator/** - Orchestrator abstraction with provider pattern
- **orchestrator/swarm/** - Docker Swarm implementation (services, networks, configs, secrets)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pgEdge/control-plane](https://github.com/pgEdge/control-plane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
