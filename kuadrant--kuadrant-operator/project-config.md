---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kuadrant is a Kubernetes operator that extends Gateway API providers (Istio, Envoy Gateway) with additional policies for TLS, DNS, authentication/authorization, and rate limiting. It leverages Gateway API Policy Attachment (GEP-713) to provide a consistent policy-driven approach to traffic management.

## Build and Test Commands

### Building
```bash
# Build operator binary
make build

# Build Docker image
make docker-build

# Build with specific version
make build VERSION=1.0.0
```

### Testing
```bash
# Run unit tests
make test-unit

# Run specific unit test
make test-unit TEST_NAME=TestLimitIndexEquals

# Run specific subtest
make test-unit TEST_NAME=TestLimitIndexEquals/empty_indexes_are_equal

# Run with verbose output
make test-unit VERBOSE=1

# Run integration tests for specific environment
make test-integration GATEWAYAPI_PROVIDER=istio
make test-integration GATEWAYAPI_PROVIDER=envoygateway

# Run bare k8s integration tests (no gateway provider)
make test-bare-k8s-integration

# Run gatewayapi integration tests (GatewayAPI CRDs only)
make test-gatewayapi-env-integration

# Run Istio-specific integration tests
make test-istio-env-integration

# Run EnvoyGateway-specific integration tests
make test-envoygateway-env-integration
```

### Linting and Code Quality
```bash
# Run linter
make run-lint

# Format code
make fmt

# Run go vet
make vet

# Run all verification checks
make verify-all

# Run comprehensive pre-commit checks
make pre-commit

# Run pre-commit with integration tests
make pre-commit INTEGRATION_TEST_ENV=istio
make pre-commit INTEGRATION_TEST_ENV=all
```

### Local Development
```bash
# Setup local Kind cluster with all dependencies and operator
make local-setup

# Setup local environment without running operator (for local development)
make local-env-setup

# Run operator locally (after local-env-setup)
make run

# Cleanup local environment
make local-cleanup

# Install CRDs
make install

# Uninstall CRDs
make uninstall

# Deploy operator to existing cluster
make deploy

# Deploy dependencies only
make deploy-dependencies

# Local setup with a custom wasm-shim and extra extensions (e.g. threat-policy)
# 1. Build wasm-shim from local repo
docker build -t quay.io/kuadrant/wasm-shim:dev /path/to/wasm-shim
# 2. Build operator with extra extensions, load images, and deploy
make docker-build IMG=quay.io/kuadrant/kuadrant-operator:dev EXTRA_EXTENSIONS=threat-policy
make local-setup IMG=quay.io/kuadrant/kuadrant-operator:dev
# 3. Push wasm-shim to Kind's local registry (Istio fetches wasm via OCI, not container runtime)
docker tag quay.io/kuadrant/wasm-shim:dev localhost:5001/kuadrant/wasm-shim:dev
docker push localhost:5001/kuadrant/wasm-shim:dev
# 4. Point operator at the local registry (use the registry's Kind-network IP)
REGISTRY_IP=$(docker inspect kind-registry --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}')
kubectl set env deployment/kuadrant-operator-controller-manager -n kuadrant-system \
  RELATED_IMAGE_WASMSHIM=oci://${REGISTRY_IP}:5000/kuadrant/wasm-shim:dev
```

### Code Generation
```bash
# Generate manifests (CRDs, RBAC, webhooks)
make manifests

# Generate code (DeepCopy methods)
make generate

# Generate extension manifests
make extensions-manifests

# Update dependency manifests
make dependencies-manifests
```

## Architecture

### Policy System

Kuadrant implements a sophisticated policy attachment system based on Gateway API's Policy Attachment (GEP-713). The operator manages five core policies:

- **AuthPolicy** (`api/v1/authpolicy_types.go`): Authentication and authorization via Authorino
- **RateLimitPolicy** (`api/v1/ratelimitpolicy_types.go`): Rate limiting via Limitador
- **DNSPolicy** (`api/v1/dnspolicy_types.go`): DNS and load balancing configuration
- **TLSPolicy** (`api/v1/tlspolicy_types.go`): TLS certificate management via cert-manager
- **TokenRateLimitPolicy** (`api/v1alpha1/tokenratelimitpolicy_types.go`): Token-based rate limiting for AI/LLM workloads

Policies attach to Gateway API resources (Gateway, HTTPRoute, GRPCRoute) using `targetRef` fields and are reconciled through a workflow-based controller system.

### Extensions System

The operator supports out-of-process (OOP) extensions via gRPC over Unix domain sockets. Extensions are separate controller processes that communicate with the main operator to:

1. **Evaluate CEL expressions** with access to Kuadrant's topology (Gateway, Routes, Policies)
2. **Publish data bindings** that influence downstream policy configurations (AuthConfig, Limitador, Envoy wasm)
3. **Subscribe to cluster events** for reactive reconciliation

**Extension Architecture:**
- Extensions live in `cmd/extensions/*/` (e.g., `oidc-policy`, `plan-policy`, `telemetry-policy`)
- They connect to the operator via Unix socket (path provided as first CLI arg)
- Communication uses gRPC protocol defined in `pkg/extension/`
- Extensions use the SDK in `pkg/extension/` for controller building

**Data Flow:**
```
Extension Controller → kuadrant.Resolve(CEL) → Operator Policy Machinery
                    ↓

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kuadrant/kuadrant-operator](https://github.com/Kuadrant/kuadrant-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
