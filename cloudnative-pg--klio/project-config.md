---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project overview

Klio is an enterprise-grade backup and recovery manager for PostgreSQL databases.
It has special integrations for CloudNativePG on Kubernetes, but should
work with any PostgreSQL setup.
It consists of two main Go modules in a monorepo structure.

## Build System

This project uses [Task](https://taskfile.dev/) (not Make) as the primary build system. The main `Taskfile.yml` is in the repository root.

### Common Commands

```bash
# Run complete CI pipeline
task all:ci

# Core module
task core:lint              # Run golangci-lint on core
task core:go-test           # Run unit tests
task core:protoc-gen-go-grpc  # Compile proto files

# Operator module
task operator:lint          # Run golangci-lint on operator
task operator:controller-gen  # Generate CRDs and DeepCopy methods
task operator:helm-chart    # Generate Helm chart

# Documentation
task documentation:ci       # Run documentation CI
task documentation:spellcheck  # Run spellcheck

# Integration testing
task integration:devenv     # Create ephemeral development environment
task integration:e2e        # Run e2e tests (requires KIND_CLUSTER_NAME)

# Cleanup
task clean:all              # Full cleanup of generated elements
```

### Key Custom Resources

- **Server**: Deploys the Klio backup server (StatefulSet with PVCs for cache, data, queue)
- **PluginConfiguration**: Configures the CloudNativePG plugin for a cluster

### Build Pipeline

The CI uses Dagger for containerized builds. Key tasks in `Taskfile.yml`:
- Proto compilation via `cloudnative-pg/daggerverse/protoc-gen-go-grpc`
- Linting via `sagikazarmark/daggerverse/golangci-lint`
- Controller-gen via `cloudnative-pg/daggerverse/controller-gen`
- Container builds via `docker buildx bake`

### Unit Tests

```bash
# Core tests (uses envtest for Kubernetes)
task core:go-test

# Operator tests
cd operator && go test ./... -v
```

#### Running a Single Test
```bash
# Core module
cd core && go test -v -run TestFunctionName ./path/to/package

# Operator module
cd operator && go test -v -run TestFunctionName ./path/to/package
```

### E2E Tests
```bash
# Requires a Kind cluster created by CNPG hack/setup-cluster.sh
KIND_CLUSTER_NAME=$(kind get clusters  | grep pg-operator-e2e) task integration:e2e
```

#### E2E Test Feature Registration

Features can be registered with execution mode options:

```go
// Parallel execution (default)
runner.RegisterFeature(BackupFromPrimary(ns))

// Serial execution (for tests sharing resources)
runner.RegisterFeature(
    Tier2Retention(ns),
    runner.WithSerialExecution(),
)

// Register multiple features with same configuration
runner.RegisterFeatures(
    []runner.FeatureOption{runner.WithSerialExecution()},
    feat1,
    feat2,
)
```

**Execution Order:**
1. All parallel features run concurrently.
2. Serial features run sequentially after parallel features complete.
3. Use serial execution for tests that share infrastructure.

> **Important:** When adding a new e2e test, update the "Test
> Structure" section in
> `documentation/web/docs/developer/running-e2e-tests.md` to list the
> new file and its feature function(s).

#### Test Package Structure

The `operator/test/machinery` package may only contain helpers that are
generic to any Kubernetes cluster or to CloudNativePG. Anything
Klio-specific (Server/PluginConfiguration resources, Klio config,
Klio-only assertions) must live outside `machinery` — e.g. under
`operator/test/klio` or the `operator/test/e2e` package itself.

## Code Style

- Avoid inline error strings; define error variables instead
  (e.g., `var ErrSomething = errors.New("message")`)
- Comments on exported functions and variables must end with a period.
- Test function names must match `^(_|[a-zA-Z0-9]+)$` — no underscores
  (e.g., `TestGetStatusEmpty` not `TestGetStatus_Empty`).

# Important notes

- These files must be kept in sync:
  - `operator/pkg/config/server.go` ↔ `core/pkg/config/server.go`
  - `operator/pkg/config/client.go` ↔ `core/pkg/config/client.go`

- When you change a metric in `core/internal/opentelemetry/catalog.go`
  (rename, add, remove, or change a metric's unit, type, or attributes),
  update the Grafana dashboard builder in `observability/grafana/` to match
  and regenerate the committed JSON with `task grafana:gen`. The builder
  references the **Prometheus** export names of these metrics, where the unit
  and instrument type add suffixes (so a unit change shifts the suffix too).
  `gcx` lint validates PromQL syntax and units but cannot catch a query that
  references a metric the code no longer emits.

## Architecture notes

### Client configuration

The `ClientConfig` struct contains configuration for both Kopia (base backups)
and gRPC (WAL streaming) clients:

- `ClientConfig.ClusterName` - shared cluster identifier (must match certificate
  CN hostname)
- `ClientConfig.Base` - Kopia repository client config (for base backups)
- `ClientConfig.Wal` - gRPC WAL client config (for WAL streaming)

The Kopia client validates that `ClusterName` matches the hostname in the client
certificate's Common Name (format: `userName@hostName`). This prevents silent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudnative-pg/klio](https://github.com/cloudnative-pg/klio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
