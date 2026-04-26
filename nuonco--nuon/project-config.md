---
trigger: always_on
description: This is a comprehensive monorepo for Nuon, a BYOC (Bring Your Own Cloud) platform that helps software vendors deploy applications to their customers' cloud accounts.
---

# Nuon Monorepo Overview

This is a comprehensive monorepo for Nuon, a BYOC (Bring Your Own Cloud) platform that helps software vendors deploy applications to their customers' cloud accounts.

## Repository Structure

This monorepo is written primarily in **Go** (main module: `github.com/nuonco/nuon`) with several TypeScript/JavaScript projects for UI components.

### Core Directories

#### `/bins/` - Command Line Tools & Executables
Binaries compiled and run as executables (not deployed as Kubernetes services):
- **`cli/`** - Public-facing CLI tool for developers (`nuon` command)
- **`nuonctl/`** - Internal CLI with 80+ operational scripts and dev tooling
- **`runner/`** - Deployment execution binary (runs in customer K8s + VMs)

#### `/services/` - Web Services & Applications
- **`ctl-api/`** - Control API service (Go)
- **`dashboard-ui/`** - Main dashboard UI: the running app is a client-side SPA served by a Go server from the `client/` directory (not the `src/` Next.js app)
- **`website-v2/`** - Marketing website (Astro)
- **`website/`** - Legacy website (Astro)
- **`wiki/`** - Internal documentation site (Astro + Starlight)
- **`e2e/`** - End-to-end testing service
- **`orgs-api/`** - Organizations API service (DEPRECATED)
- **`workers-*`** - Various worker services for background tasks
  - **`workers-executors/`** - (DEPRECATED) Previously core execution engine
  - **`workers-canary/`** - Canary testing service
  - **`workers-infra-tests/`** - Infrastructure testing service

#### `/pkg/` - Shared Go Libraries
Core shared packages used across Go services:
- `api/` - API client libraries
- `config/` - Configuration management
- `helm/` - Helm chart operations
- `kube/` - Kubernetes utilities
- `terraform/` - Terraform operations
- `metrics/` - Metrics and telemetry
- `temporal/` - Temporal workflow engine integration

#### `/infra/` - Infrastructure as Code
Terraform modules and configurations for:
- AWS infrastructure
- Azure support
- Kubernetes (EKS) clusters
- Monitoring (DataDog)
- DNS management
- Developer environments

#### `/charts/` - Helm Charts
Kubernetes Helm charts for various components:
- `common/` - Shared chart templates
- `temporal/` - Temporal workflow engine
- Application-specific charts

#### `/docs/` - Documentation
- API reference documentation
- User guides and tutorials
- Platform support documentation
- Production readiness guides

#### `/seed/` - Example Applications
Template applications and configurations for different deployment scenarios:
- EKS examples
- Azure AKS examples
- Component-based applications
- Various infrastructure patterns

#### `/exp/` - Experimental Features
Development and testing area for new features and proof-of-concepts

#### `/graveyard/` - Deprecated Code
Archive of deprecated code and components

#### `/images/` - Container Images
Dockerfiles and build configurations for various container images

#### `/wiki/` - Internal Company Wiki
Team documentation, processes, and company information

## Key Technologies

- **Backend**: Go 1.25+
- **Frontend**: Next.js (React), Astro
- **Infrastructure**: Terraform, Kubernetes, Helm
- **Cloud Platforms**: AWS (primary), Azure, GCP
- **Workflow Engine**: Temporal
- **Monitoring**: DataDog, OpenTelemetry
- **Databases**: PostgreSQL, ClickHouse

## Common Commands

Based on the repository structure, these commands are likely useful:

### Development
```bash
# CLI development
cd bins/cli && go run main.go

# Dashboard UI development  
cd services/dashboard-ui && npm run dev

# Wiki development
cd services/wiki && npm run dev
```

### Infrastructure
```bash
# Terraform operations
cd infra/[module] && terraform plan
```

### Testing
```bash
# Go tests
go test ./...

# Frontend tests
cd services/dashboard-ui && npm test
```

### Code Generation

Use `go generate` to regenerate code after making changes:

```bash
# Temporal activities (ctl-api)
go generate ./services/ctl-api/...

# Temporal workflow types (executors)
go generate ./pkg/workflows/types/executors/...

# SDKs (nuon-go and nuon-runner-go)
go generate ./sdks/nuon-go/...
go generate ./sdks/nuon-runner-go/...

# Regenerate all code in the monorepo
go generate ./...
```

**When to run code generation:**
- After adding/modifying Go struct types (especially in `/internal/app/`)
- After adding `@temporal-gen` annotations to functions
- After changing API endpoint definitions with swagger annotations
- When you see compilation errors about missing generated files
- When generated files (`.activity_gen.go`, `.workflow_gen.go`, swagger docs) are out of sync

**Key `go:generate` locations:**
- `services/ctl-api/main.go` - API types and swagger docs
- `pkg/workflows/types/executors/gen.go` - Temporal activity/workflow generation
- `sdks/nuon-go/client.go` - Go SDK client generation
- `sdks/nuon-runner-go/client.go` - Runner SDK client generation

## Running Services Locally

Ask the user how to start the services to test your changes.

### Expected URLs

```
Frontend Services:
- Dashboard UI:     http://localhost:4000
- Wiki:             http://localhost:4321

Backend Services:
- CTL-API Public:   http://localhost:8081

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nuonco/nuon](https://github.com/nuonco/nuon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
