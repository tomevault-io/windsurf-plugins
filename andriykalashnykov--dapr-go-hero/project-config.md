---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Educational Go application demonstrating Dapr building blocks (Pub/Sub with routing, Secret Store, State Store, Service Invocation) with four client approaches: custom HTTP, custom gRPC, Go SDK HTTP, and Go SDK gRPC.

## Build, Lint & Test Commands

```bash
make help               # List all available Make targets
make deps               # Install all CLI tools pinned in .mise.toml via mise
make clean              # Remove build artifacts
make static-check       # Composite quality gate: lint-ci + lint + sec + vulncheck + secrets + docker-lint + trivy-fs + trivy-config + mermaid-lint + diagrams-check + deps-prune-check
make lint               # Run golangci-lint (includes gocritic, gosec via .golangci.yml)
make lint-ci            # Lint GitHub Actions workflows (actionlint + shellcheck)
make sec                # Run gosec security scanner (excludes generated proto/ dir)
make vulncheck          # Check dependencies for known vulnerabilities (govulncheck)
make secrets            # Scan for hardcoded secrets (gitleaks)
make trivy-fs           # Filesystem scan for CVEs / secrets / misconfigs (CRITICAL, HIGH)
make trivy-config       # K8s manifest scan for security misconfigurations (KSV-*)
make test               # Run unit tests (go test -race -v ./...)
make integration-test   # Run integration tests (real PostgreSQL + Dapr-state stub + gRPC server)
make build              # Compile both binaries (depends on deps only)
make format             # Auto-format Go source (golangci-lint --fix + gofmt)
make proto-gen          # Regenerate gRPC code from .proto files (mise-pinned protoc)
make update             # Update all deps to latest (go get -u ./... && go mod tidy)
make ci                 # Full CI pipeline: deps → format → static-check → test → integration-test → build
make ci-run             # Run GitHub Actions workflow locally via act
make deps-check         # Show Go version and mise tool status
make deps-prune         # Remove unused and redundant dependencies
make deps-prune-check   # Verify no prunable dependencies (CI gate)
make release            # Tag and push a release (runs full build first)
make docker-build       # Build container images (inventory + products)
make docker-lint        # Lint Dockerfiles with hadolint
make docker-smoke-test  # Boot each image briefly and assert it stays up (Gate 3 of /harden-image-pipeline; SERVICES=name to scope)
make kind-up            # Full stack: create KinD cluster + cloud-provider-kind + Dapr + deploy manifests (alias for kind-deploy)
make kind-down          # Full teardown (alias for kind-destroy)
make kind-create        # Cluster + LoadBalancer controller + Dapr (no app deploy)
make kind-bootstrap     # Install cloud-provider-kind + Dapr into an existing cluster (CI use after helm/kind-action)
make kind-deploy        # kind-create + k8s-deploy (full stack up)
make kind-destroy       # Delete cluster, stop cloud-provider-kind, prune kindccm-* orphan sidecars
make k8s-deploy         # Build images, load into KinD, deploy all manifests
make k8s-undeploy       # Remove all K8s manifests
make k8s-status         # Show pod/service status across all namespaces
make e2e                # Run end-to-end tests (default: sdk-http mode)
make e2e-all            # Run e2e across all 4 client modes (sdk-http, sdk-grpc, custom-http, custom-grpc)
make e2e-setup          # Alias for kind-deploy (CI compatibility)
make e2e-teardown       # k8s-undeploy + kind-destroy (CI compatibility)
```

Run a single test:
```bash
go test ./pkg/features/widgets/... -run TestName -v
```

## Three-Layer Test Pyramid

| Layer | Command | Latency | Scope |
|-------|---------|---------|-------|
| Unit | `make test` | seconds | Pure logic, mocked interfaces, `_test.go` files |
| Integration | `make integration-test` | tens of seconds | Real PostgreSQL via Testcontainers, stub Dapr sidecars, ephemeral-port gRPC servers; gated by `//go:build integration` |
| E2E | `make e2e` / `make e2e-all` | minutes | KinD cluster + cloud-provider-kind + Dapr + full manifest deploy + 11 assertions across 4 client modes |

All three are wired into CI as separate jobs. The unit/integration split keeps the inner-loop fast; the integration layer only runs when changes affect code paths that touch external services.

## Running the Application

Prerequisites: `dapr init` (provides Redis), PostgreSQL container, `widgets` table from `tables.sql`.

```bash
# Terminal 1: Products gRPC service
make run-products

# Terminal 2: Inventory service (pick one client mode)
make run-custom-http    # Custom HTTP client (Fiber), app port 3001
make run-custom-grpc    # Custom gRPC client, app port 4001
make run-sdk-http       # Dapr Go SDK HTTP, app port 3002
make run-sdk-grpc       # Dapr Go SDK gRPC, app port 4002

# Terminal 3: Publish events
make send-widget        # → PostgreSQL via widgets service
make send-gadget        # → Redis State Store via gadgets service
make send-thingamajig   # → gRPC Products service
make send-all           # All three

# Query REST API (Fiber on port 3000)
make get-widget         # GET /v1/widgets/widget

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndriyKalashnykov/dapr-go-hero](https://github.com/AndriyKalashnykov/dapr-go-hero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
