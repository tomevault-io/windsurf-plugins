---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Distr is an open-source software distribution platform that enables companies to distribute applications to self-managed customers.
It provides centralized management of deployments, artifacts, agents, licenses, and includes an OCI-compatible container registry.
The platform consists of a control plane (Hub) running in the cloud and agents that run in customer environments.

## Architecture

### High-Level Components

1. **Distr Hub** (`cmd/hub/`): The main control plane server
   - Go backend with chi router
   - Angular frontend (TypeScript, TailwindCSS 4)
   - REST API at `/api/v1`
   - Serves the compiled frontend on root path

2. **Agents** (`cmd/agent/`):
   - `docker/`: Docker agent for managing Docker Compose deployments
   - `kubernetes/`: Kubernetes agent for managing Helm deployments
   - Agents connect to Hub, collect logs/metrics, execute deployments

3. **SDK** (`sdk/js/`): JavaScript/TypeScript SDK for interacting with Distr API

### SDK Architecture (TypeScript)

The SDK is a standalone subproject in `sdk/js/` with its own package.json, dependencies, and build process.

- **Location**: `sdk/js/`
- **Package**: `@distr-sh/distr-sdk`
- **Package Manager**: pnpm
- **Build**: `pnpm build` (compiles TypeScript to `dist/`)
- **Test**: `pnpm test:examples` (runs example test client)
- **Examples**: `sdk/js/src/examples/` contains usage examples
- **Main classes**:
  - `Client`: Low-level API client (in `src/client/client.ts`)
  - `DistrService`: High-level service with convenience methods (in `src/client/service.ts`)

When working with the SDK:

- Always build the SDK with `mise build:sdk` after making changes
- Use pnpm (not npm) for all package management
- Use `DistrService` for high-level operations (preferred)
- Use `Client` for direct API access when needed
- Example files use a config from `src/examples/config.ts`

### Backend Architecture (Go)

- **Database**: PostgreSQL accessed via pgx/v5 with connection pooling
- **Router**: chi/v5 with middleware-based architecture
- **Authentication**: JWT-based with support for OIDC, API keys, and agent tokens
- **OCI Registry**: Adapted from google/go-containerregistry for serving Docker images, Helm charts, and other artifacts
- **Storage**: S3-compatible object storage (MinIO for dev) for registry blobs
- **Migrations**: SQL migrations in `internal/migrations/sql/` managed by golang-migrate
- **Database queries**: All database interactions are in `internal/db/` with transaction support

Key internal packages:

- `internal/handlers/`: HTTP request handlers
- `internal/routing/`: Route configuration and middleware setup
- `internal/authn/`: Authentication providers (JWT, API keys, agent tokens)
- `internal/db/`: Database queries and models
- `internal/registry/`: OCI registry implementation
- `internal/middleware/`: HTTP middleware (logging, auth, Sentry, etc.)
- `internal/svc/`: Business logic services
- `internal/mapping/`: Mapping logic for data transformations between DTOs and domain models
- `api/`: All request structs used by HTTP handlers should be in the api package and not in the handler package

### Frontend Architecture (Angular)

- **Framework**: Angular with standalone components
- **Styling**: TailwindCSS 4, SCSS, Flowbite components
- **Routing**: Angular Router with lazy-loaded routes
- **State**: Service-based state management
- **Forms**: Reactive forms with Angular Forms
- **Key directories**:
  - `frontend/ui/src/app/`: All application components
  - `frontend/ui/src/app/services/`: Data services and API clients
  - `frontend/ui/src/app/components/`: Reusable UI components
  - `frontend/ui/src/buildconfig/`: Build-time configuration injected by Go

The frontend is built into `internal/frontend/dist/ui/` and served by the Go backend.

### Database Schema

The database schema is managed through SQL migrations in `internal/migrations/sql/`. Key tables include:

- `user_accounts`: User authentication and profiles
- `organizations`: Multi-tenant organizations
- `deployments`: Application deployments
- `deployment_targets`: Customer environments (agents)
- `artifacts`: Software artifacts (Docker images, Helm charts)
- `applications`: Artifact collections
- `deployment_log_records`: Logs from deployments
- `licensekey`: License keys that vendors can generate for its customers
- `application_entitlements` & `artifact_entitlements`: Access entitlements for applications and artifacts

This database stores timestamps as `TIMESTAMP` (without time zone), not `TIMESTAMPTZ`.

## Common Commands

### Building

````bash
# Build hub (includes frontend build)
mise run build:hub:community        # Community edition

# Build agents
mise run build:agent:docker
mise run build:agent:kubernetes

Binaries are output to `dist/`.

### Linting and Formatting

```bash
# Auto-fix linting issues
mise run format              # All
mise run format:go           # Go only
mise run format:frontend     # Frontend only
````

Go linting uses golangci-lint with config in `.golangci.yml`. Frontend uses Prettier with config in `.prettierrc.mjs`.

## Code Patterns and Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [distr-sh/distr](https://github.com/distr-sh/distr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
