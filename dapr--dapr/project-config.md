---
trigger: always_on
description: This is **dapr/dapr**, the Go implementation of the [Dapr](https://dapr.io) distributed application runtime. Dapr is a graduated CNCF project that provides APIs (state management, pub/sub, service invocation, actors, workflows, bindings, secrets, configuration, distributed lock, cryptography) as a sidecar process alongside applications.
---

# Dapr Runtime – Copilot Coding Agent Instructions

## Repository Overview

This is **dapr/dapr**, the Go implementation of the [Dapr](https://dapr.io) distributed application runtime. Dapr is a graduated CNCF project that provides APIs (state management, pub/sub, service invocation, actors, workflows, bindings, secrets, configuration, distributed lock, cryptography) as a sidecar process alongside applications.

**Go module**: `github.com/dapr/dapr`  
**Go version**: See `go.mod` (currently 1.26.x)  
**License**: Apache 2.0

---

## Repository Layout

```
cmd/                  # Main entry points for each binary
  daprd/              # Dapr sidecar runtime (primary binary)
  injector/           # Kubernetes sidecar injector
  operator/           # Kubernetes operator
  placement/          # Actor placement service
  scheduler/          # Scheduler service
  sentry/             # Certificate authority / mTLS
pkg/                  # Core library packages
  actors/             # Virtual actor framework
  api/                # HTTP and gRPC API implementations
    grpc/             # gRPC server
    http/             # HTTP server
    universal/        # Shared handler logic
  components/         # Component registry and loaders
  config/             # Configuration parsing
  messaging/          # Service-to-service invocation
  middleware/         # HTTP and gRPC middleware
  operator/           # Kubernetes operator logic
  placement/          # Placement client and service
  resiliency/         # Retry/circuit-breaker/timeout policies
  runtime/            # Main runtime initialization
  scheduler/          # Scheduler logic
  security/           # mTLS and SPIFFE
  sentry/             # Sentry CA service
dapr/proto/           # Protobuf definitions
  runtime/            # Dapr runtime proto
  common/             # Common proto types
  operator/           # Operator proto
  placement/          # Placement service proto
  scheduler/          # Scheduler service proto
  sentry/             # Sentry proto
tests/
  integration/        # Integration tests (run against real daprd binary)
  e2e/                # End-to-end tests (require Kubernetes)
  perf/               # Performance tests
.build-tools/         # Internal build tools
charts/               # Helm charts
```

---

## Building

**Build all binaries** (output to `dist/{os}_{arch}/release/`):
```sh
make build
```

**Build a specific binary directly** (faster, good for iteration):
```sh
cd cmd/daprd
go build -tags=allcomponents -v
```

**Build tags** for the daprd sidecar:
- `allcomponents` (default) — includes all components
- `stablecomponents` — includes only stable components

> The `DAPR_SIDECAR_FLAVOR` variable in the Makefile controls which build tag is applied.

**Cross-compile**:
```sh
make build GOOS=linux GOARCH=amd64
```

---

## Testing

### Unit Tests

Unit tests live alongside source code in `pkg/`, `utils/`, and `cmd/`. They use the build tag `//go:build unit`.

**Run all unit tests** (requires `gotestsum`; see below):
```sh
make test
```

**Run a single package** (no `gotestsum` needed):
```sh
go test -tags=unit,allcomponents ./pkg/actors/...
```

**Run a specific test**:
```sh
go test -tags=unit,allcomponents -run TestFoo ./pkg/actors/...
```

**Install `gotestsum`** (required by `make test`):
```sh
go install gotest.tools/gotestsum@latest
```

### Integration Tests

Integration tests in `tests/integration/` run against the compiled daprd binary. They use the build tag `integration`.

**Run integration tests**:
```sh
make test-integration
```

**Run a specific integration test**:
```sh
make test-integration ARGS="-run TestSuiteName/TestName"
```

**Run in parallel mode**:
```sh
make test-integration-parallel
```

> Integration tests require `CGO_ENABLED=1`.

### E2E Tests

E2E tests require a running Kubernetes cluster with Dapr installed. See `tests/docs/running-e2e-test.md` for details. These are not normally run locally; CI handles them.

---

## Linting

```sh
make lint
```

Uses `golangci-lint` **v2.10.1** with build tags `allcomponents,subtlecrypto`. Always use this exact version to avoid false errors. Download from: https://github.com/golangci/golangci-lint/releases/tag/v2.10.1

**Auto-fix issues**:
```sh
make lint-fix
```

---

## Formatting and Tidy

**Format all code and tidy go.mod**:
```sh
make format
```

This runs `gofumpt`, `goimports` (with local prefix `github.com/dapr/`), and `go mod tidy`.

**Tidy go.mod only**:
```sh
make modtidy
```

**One-line local check** (format + test + lint + verify no uncommitted changes):
```sh
make check
```

---

## Code Conventions

### License Header

Every new Go file **must** start with this Apache 2.0 header:

```go
/*
Copyright <YEAR> The Dapr Authors
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
    http://www.apache.org/licenses/LICENSE-2.0

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dapr/dapr](https://github.com/dapr/dapr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
