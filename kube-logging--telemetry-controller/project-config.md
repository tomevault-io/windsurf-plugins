---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Kubernetes operator that provides multi-tenant telemetry data collection and routing. It deploys and configures:
- **OpenTelemetry Collector** (DaemonSet) — collects logs, metrics, and traces from containers
- **Routing infrastructure** — tenant-based isolation and subscription-driven data routing

Cluster operators define `Collector` and `Tenant` CRDs to set up multi-tenant isolation. Users define `Subscription` and `Output` CRDs to route telemetry data to destinations via OTLP, Fluentforward, or local file outputs.

## Project Structure

```
telemetry-controller/
├── .github/                               # GitHub Actions workflows
├── api/telemetry/v1alpha1/                # CRD type definitions
│   ├── common.go                          # Shared types (NamespacedName helper)
│   ├── collector_types.go                 # Global OTEL Collector DaemonSet configuration
│   ├── tenant_types.go                    # Routing rules and namespace selectors
│   ├── subscription_types.go              # User-defined data selection and outputs
│   ├── output_types.go                    # Output types (OTLP, file, fluentforward, etc.)
│   ├── bridge_types.go                    # Cross-tenant routing
│   └── otlp_config.go                     # OTLP component configuration models (mainly used for outputs)
├── charts/telemetry-controller/           # Helm chart (CRDs synced via make manifests)
├── cmd/main.go                            # Operator entry point (controller manager setup)
├── controllers/telemetry/
│   ├── collector_controller.go            # Manages Collector CR and OTEL configuration
│   └── route_controller.go                # Manages Tenants and watches Subscriptions/Outputs
├── e2e/                                   # KIND-based bash e2e test suites
└── pkg/
    ├── resources/
    │   ├── manager/                       # CR manager abstractions (extends BaseManager)
    │   │   ├── bridge_manager.go
    │   │   ├── collector_manager.go
    │   │   ├── manager.go                 # CRUD operations and logging
    │   │   └── tenant_resource_manager.go # Collector-specific orchestration
    │   ├── otel_conf_gen/                 # OpenTelemetry Collector config generator
    │   │   ├── pipeline/
    │   │   │   └── components/            # Receivers, processors, connectors, exporters
    │   │   ├── otel_conf_gen.go           # OTEL config generation entry point
    │   │   └── validator/                 # Configuration validation
    │   └── problem/                       # Problem/issue tracking for status conditions
    └── sdk/
        ├── model/
        └── utils/                         # Helper functions
```

## Module Structure

The repository contains a single Go module (`go.mod`) for the operator binary, controllers, API types, and resource generators. All types are in `api/telemetry/v1alpha1/`.

## Build & Development Commands

```bash
# Build operator binary
make build

# Verify no uncommitted generated files (useful in CI)
make check-diff

# Full generation cycle after any API type change (codegen + fmt + manifests)
make generate

# Format all code
make fmt

# Run go vet
make vet

# Run all linters (golangci-lint)
make lint
make lint-fix   # auto-fix issues

# Run all tests
make test

# Run a single test
go test -run TestName -v ./controllers/telemetry/...
# or for pkg tests:
go test -run TestName -v ./pkg/resources/...

# Run e2e tests on KIND cluster
make test-e2e

# Install CRDs into cluster
make install

# Remove CRDs from cluster
make uninstall

# Run operator locally (hot-reload during development)
make run

# Build and deploy to cluster
make docker-build IMG=telemetry-controller:latest
kind load docker-image telemetry-controller:latest
make deploy IMG=telemetry-controller:latest

# Remove controller from cluster
make undeploy


# Run with delve debugger (remote debug on :2345)
make run-delve
```

## Architecture

### CRD API (`api/telemetry/v1alpha1/`)

Core resource types:
- **`Collector`** — cluster-scoped root resource; defines global OTEL Collector DaemonSet settings and tenant selection via `tenantSelector`
- **`Tenant`** — cluster-scoped routing rules with namespace selectors:
  - `logSourceNamespaceSelectors` — where logs originate
  - `subscriptionNamespaceSelectors` — where users can create subscriptions
  - `routeConfig` — routing connector configuration (default pipelines, error handling)
  - `persistence` — optional file storage configuration for buffering
- **`Bridge`** — cluster-scoped advanced routing for cross-tenant scenarios
- **`Subscription`** — namespace-scoped data selection rules with output references; users create these in their tenant's allowed namespaces
- **`Output`** — namespace-scoped telemetry destinations; see `api/telemetry/v1alpha1/output_types.go` for supported exporter types

### Controllers (`controllers/telemetry/`)

- **`CollectorReconciler`** — primary reconciler; orchestrates:
  - OTEL Collector DaemonSet creation/update
  - ConfigMap generation from Tenant/Subscription/Output routing rules
  - RBAC (ServiceAccount, ClusterRole, ClusterRoleBinding)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kube-logging/telemetry-controller](https://github.com/kube-logging/telemetry-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
