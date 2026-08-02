---
trigger: always_on
description: This file provides guidance to AI assistants and coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI assistants and coding agents when working with code in this repository.

## Project Overview

Authorino is a Kubernetes-native authorization service that implements Envoy's external authorization gRPC protocol. It acts as an authorization layer between Envoy proxy and upstream services, providing hybrid API security with support for multiple authentication protocols (JWT/OIDC, API keys, mTLS, OAuth2 token introspection, Kubernetes TokenReview) and authorization mechanisms (pattern-matching, OPA/Rego, Kubernetes SubjectAccessReview, SpiceDB).

## Repository Map

```text
authorino/
├── api/
│   └── v1beta3/          # CRD type definitions
│       └── auth_config_types.go  # Primary AuthConfig spec
├── controllers/
│   ├── auth_config_controller.go # Reconciliation: AuthConfig watcher
│   └── secret_controller.go      # Secret watcher and Secret-triggered reconciliation
├── pkg/
│   ├── auth/             # Pipeline interfaces & credential extraction
│   ├── evaluators/
│   │   ├── identity/     # Authentication evaluators (JWT, API key, OAuth2, mTLS, etc.)
│   │   ├── metadata/     # External metadata fetchers (HTTP, OIDC UserInfo, UMA)
│   │   ├── authorization/ # Authorization evaluators (OPA, pattern-matching, K8s SAR, SpiceDB)
│   │   └── response/     # Dynamic response builders (wristbands, JSON injection)
│   ├── index/            # In-memory host→AuthConfig index
│   ├── metrics/          # Prometheus metrics
│   └── service/          # gRPC and HTTP authorization service implementations
├── docs/                 # Architecture, features, user guides
├── tests/                # Integration and e2e tests
├── hack/                 # Build/dev scripts
├── deploy/               # Kubernetes manifests to deploy Authorino (operator)
├── install/              # Main Kustomize files to install Authorino, CRDs and RBAC
└── main.go               # Entry point: server / webhooks / version commands
```

## Essential Commands

### Development Workflow

```bash
# Download dependencies
make vendor

# Run tests
make test

# Run benchmarks
make benchmarks

# Build the binary
make build

# Build Docker image
make docker-build

# Run linter
make lint

# Fix linting issues
make lint-fix
```

### Code Generation

```bash
# Generate deepcopy code for types
make generate

# Generate CRDs and RBAC manifests
make manifests
```

### Local Development with Kind

```bash
# Full local setup (cluster + deps + build + deploy + apps)
make local-setup

# Rebuild and redeploy after code changes
make local-rollout

# Delete local cluster
make local-cleanup

# Port forward to Envoy for testing
kubectl port-forward deployment/envoy 8000:8000
```

### Running Single Tests

```bash
# Run specific test
go test ./pkg/auth -run TestSpecificFunction

# Run with verbose output
go test -v ./pkg/evaluators/...

# Run with race detection
go test -race ./...
```

### Static Analysis

```bash
# Verify GitHub Actions are pinned to commit SHAs
make verify-ratchet

# Pin GitHub Actions to commit SHAs
make ratchet-pin
```

## Architecture

### Core Components

**Auth Pipeline**: Authorino processes requests through a 5-phase pipeline:
1. **Authentication** (phase i): Verify identity from credentials (at least one must succeed)
2. **Metadata** (phase ii): Fetch external data to enrich context
3. **Authorization** (phase iii): Evaluate policies (all must pass)
4. **Response** (phase iv): Build dynamic responses (headers, wristbands, dynamic metadata)
5. **Callbacks** (phase v): Send HTTP callbacks

**Index**: In-memory data structure that maps hosts to AuthConfig specs. Built and reconciled by watching AuthConfig and Secret resources.

**Authorization JSON**: Working memory for each request containing `context` (request data from Envoy) and `auth` (resolved identity, metadata, authorization results). Evaluators read/write from this structure.

### Key Packages

- `api/v1beta3`: AuthConfig CRD definitions
- `controllers/`: Kubernetes controllers for AuthConfig and Secret reconciliation
- `pkg/service/`: gRPC and HTTP authorization service implementations
- `pkg/auth/`: Core authentication logic and credential extraction
- `pkg/evaluators/`: Implementations of auth pipeline phases:
  - `evaluators/identity/`: Authentication evaluators (JWT, API key, OAuth2, etc.)
  - `evaluators/authorization/`: Authorization evaluators (OPA, pattern-matching, K8s SAR, etc.)
  - `evaluators/metadata/`: External metadata fetchers (HTTP, OIDC UserInfo, UMA)
  - `evaluators/response/`: Dynamic response builders (wristbands, JSON injection)
- `pkg/index/`: AuthConfig index management
- `pkg/metrics/`: Prometheus metrics
- `pkg/trace/`: OpenTelemetry tracing

### Main Entry Point

`main.go` defines a Cobra CLI with three commands:
- `authorino server`: Runs the authorization server (primary mode)
- `authorino webhooks`: Runs validation webhooks
- `authorino version`: Prints version info

The server command starts:
- gRPC auth service (port 50051, implements Envoy external auth protocol)
- HTTP auth service (port 5001, raw HTTP interface)
- OIDC server (port 8083, for Festival Wristband token discovery)
- Kubernetes reconciliation managers (AuthConfig & Secret controllers)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kuadrant/authorino](https://github.com/Kuadrant/authorino) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
