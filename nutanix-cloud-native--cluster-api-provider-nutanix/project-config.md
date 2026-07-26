---
trigger: always_on
description: This file provides **AI coding agents** with project-specific instructions for the **Cluster API Provider Nutanix (CAPX)** repository.
---

# AGENTS.md

This file provides **AI coding agents** with project-specific instructions for the **Cluster API Provider Nutanix (CAPX)** repository.

The goals are:
- Keep changes aligned with existing controller + API patterns.
- Use the repo’s `make` targets as the source of truth for generation/lint/test steps.
- Avoid running expensive or environment-dependent workflows unless explicitly requested.

## Project Overview

This is a Kubernetes Cluster API infrastructure provider for Nutanix Cloud Infrastructure, written in Go. It enables declarative management of Kubernetes clusters on Nutanix infrastructure using the Cluster API pattern.

## Prerequisites

Only these tools need to be installed manually:
- [devbox](https://www.jetpack.io/devbox/docs/installing_devbox/) - Package management
- [direnv](https://direnv.net/docs/installation.html) - Environment management
- A container runtime: **Docker or Podman**

All other tools (Go, kubectl, Kind, clusterctl, controller-gen, etc.) are managed via devbox.

## Getting Started (Local Dev)

```bash
# Clone and enter repo
gh repo clone nutanix-cloud-native/cluster-api-provider-nutanix
cd cluster-api-provider-nutanix

# Dependencies will auto-install via devbox/direnv (may take time on first run)
# If direnv prompts, allow it:
direnv allow

# Build the project
make build

# Build container image
make docker-build

# Push container image to LOCAL_IMAGE_REGISTRY
make docker-push

# Generate manifests and code
make manifests
make generate
```

## Repo “Source of Truth”

- **Commands**: use `make help` to discover targets and rely on `Makefile` descriptions.
- **API types**: `api/v1beta1/`
- **Controllers**: `controllers/`
- **Shared packages**: `pkg/`
- **Manifests/CRDs/RBAC**: `config/`
- **Templates**: `templates/`
- **Unit tests**: co-located in packages (plus controller tests)
- **Template tests**: `templates/template_test.go` driven by `make template-test`
- **E2E tests**: `test/e2e/` (environment dependent; see below)

## Development Workflow

### Local Management Cluster (KIND)
```bash
# Create local KIND cluster
make kind-create

# Prepare local clusterctl
make prepare-local-clusterctl

# Deploy provider
make deploy
```

### Test Clusters

#### Without Topology (Traditional)
```bash
# Set required environment variables first (see developer_workflow.md)
make test-cluster-create
make generate-cluster-kubeconfig
make test-cluster-install-cni
make test-cluster-delete
```

#### With Topology (ClusterClass)
```bash
make test-cc-cluster-create
make generate-cc-cluster-kubeconfig
make test-cc-cluster-install-cni
make test-cc-cluster-delete
```

## Build / Generate / Test (Suggested Ladder)

Prefer this order; stop as soon as you have enough confidence for the change size.

### 1) Fast local correctness

- **Format**

```bash
make fmt
```

- **Static checks**

```bash
make vet
```

### 2) Code generation + manifests (when needed)

Run these when you change:
- API types / conversions / markers => `make generate`
- RBAC, CRDs, webhooks, markers => `make manifests`
- Cluster templates => `make cluster-templates`

```bash
make generate
make manifests
make cluster-templates
```

### 3) Unit tests (default)

`make unit-test` regenerates mocks as a dependency.

```bash
make unit-test
```

### 4) Template tests (heavier; still preferred over E2E)

`make template-test` builds an image and prepares local `clusterctl` overrides.

```bash
make template-test
```

### 5) E2E tests (avoid unless explicitly required)

See “E2E Testing Guidelines” below.

## Common `make` Targets

These are the ones agents should reach for first:

```bash
# Build
make build
make docker-build

# Generate
make generate
make manifests
make cluster-templates

# Test
make unit-test
make template-test

# Lint
make lint
make lint-yaml
```

## Code Style Guidelines

- Go modules with standard Go formatting (`gofmt`)
- Use `controller-gen` for CRD and RBAC generation
- Follow Kubernetes controller patterns
- Mock external dependencies for unit tests
- Use structured logging
- Add appropriate build tags for E2E tests (`//go:build e2e`)

## Testing Strategy

- **Unit Tests**: Use mocks for external dependencies (Nutanix API, Kubernetes client)
- **Template Tests**: Validate cluster template generation
- **E2E Tests**: Require actual Nutanix environment, use Ginkgo framework
- **Coverage**: Target reasonable coverage with `make coverage`

## Project Structure (High-Level)

- `/api/v1beta1/` - API types and validation
- `/controllers/` - Kubernetes controllers
- `/pkg/` - Shared packages (client, context)
- `/templates/` - Cluster templates and flavors
- `/test/e2e/` - End-to-end tests
- `/config/` - Kubernetes manifests (CRDs, RBAC, etc.)
- `/mocks/` - Generated mocks for testing

## Important Environment Variables

For development and testing, set these variables (see `docs/developer_workflow.md` for complete list):
- `NUTANIX_ENDPOINT` - Prism Central endpoint
- `NUTANIX_USER` - Nutanix username
- `NUTANIX_PASSWORD` - Nutanix password
- `NUTANIX_API_KEY` - Prism Central API key; when set, auth type is auto-selected as `api_key`, otherwise `basic_auth`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nutanix-cloud-native/cluster-api-provider-nutanix](https://github.com/nutanix-cloud-native/cluster-api-provider-nutanix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
