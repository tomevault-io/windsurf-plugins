---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## What This Is

TauGrid is a cloud-native AI infrastructure platform for GPU workloads on Kubernetes. It provides the `tau` CLI, workload queueing (Kueue), Ray cluster orchestration (KubeRay), GPU health monitoring, and observability. The repo is organized as a multi-module Go monorepo with a Python SDK.

## Build and Test Commands

The top-level Makefile provides repository-wide validation, installation, and documentation entry points:

```bash
make build                   # build first-party Go components
make test                    # run Go, offline E2E, and Python tests
make lint                    # lint Go and Python source
make check                   # build, test, lint, and check license headers
make install-tau             # install the CLI and optional Python SDK
make install-tau-cli         # install the CLI only
make install-tau-sdk         # install the Python SDK in the active Python
make install-taugrid-portal  # install the Portal CLI
make tau-docs-check          # build and validate the documentation site
```

Each component can also be built and tested from its own directory:

### CLI (`cli/`)
```bash
cd cli
make build        # produces bin/tau and bin/tau-gen
make test         # go test ./...
make lint         # go vet + gofmt + staticcheck v0.7.0
make test-kind-e2e      # Kind-based smoke test
make test-integration   # integration tests (requires built binary, 15m timeout)
```

### Core library (`core/`)
```bash
cd core
go test ./...
go vet ./...
```

### Portal (`portal/`)
```bash
cd portal
make build    # builds React assets and bin/taugrid-portal; requires Node.js 22 + npm
make test     # Go tests
make lint     # frontend typecheck + go vet + gofmt + staticcheck v0.7.0
make frontend-dev    # Vite development server
make frontend-build  # regenerate checked-in internal/portalapi/assets
```

Portal frontend source is in `portal/frontend/` (React, TypeScript, React Router,
TanStack Query). Include rebuilt embedded assets with frontend source changes;
CI checks for stale output. Direct Go commands use the checked-in assets.

### Tau core controller (`controllers/tau-core/`)
```bash
cd controllers/tau-core
make build        # produces bin/tau-core-controller
make test         # go test -race -count=1 ./...
make lint         # go vet + gofmt
make generate     # regenerate API deepcopy
make manifests    # regenerate CRDs (controller-gen v0.20.0)
make test-kind-e2e  # Kind-based integration test
```

### GPU monitoring (`monitoring/gpu-health-checker/`, `monitoring/gpu-metrics-collector/`)
```bash
cd monitoring/gpu-health-checker
make test     # CGO_ENABLED=1 go test ./...
make lint     # golangci-lint run ./...
```

### Python SDK (`sdk/python/python/`)
```bash
cd sdk/python/python
python3 -m venv .venv && source .venv/bin/activate
pip install -e '.[dev]'
python -m pytest          # run tests
python -m ruff check .    # lint (ruff 0.16.1)
```

### Portable end-to-end module (`tests/e2e/`)
```bash
cd tests/e2e
AI_RUNTIME_E2E=0 go test -count=1 ./...  # offline tests; never contacts a cluster
go vet ./...
```

The top-level `make check` always forces `AI_RUNTIME_E2E=0`. Run real-cluster
tests only when explicitly requested and only after verifying the kubeconfig:

```bash
cd tests/e2e
AI_RUNTIME_E2E=1 go test -v -timeout 15m ./...
```

### Helm Charts (`charts/`)
```bash
helm lint charts/<chart>
helm dependency build charts/<chart>
helm template test charts/<chart> --namespace taugrid-system >/dev/null

# Unit tests require the helm-unittest plugin (not built into Helm):
helm plugin install https://github.com/helm-unittest/helm-unittest.git --version v1.1.1  # Helm 3
helm plugin install oci://ghcr.io/helm-unittest/helm-unittest/unittest:1.1.1 --verify=false  # Helm 4
helm unittest charts/<chart>  # if tests/ exist
```

### Documentation site (`site/`)
```bash
cd site
make check    # builds Hugo site + runs link/content/accessibility checks
make serve    # local dev server with drafts
```

## Running a Single Test

```bash
# Go — run one test by name in any module
cd cli && go test -run TestFoo ./internal/jobrender/...
cd controllers/tau-core && go test -race -run TestReconcile ./internal/controller/...

# Python SDK
cd sdk/python/python && python -m pytest tests/test_foo.py -k test_bar
```

## CI Validation

CI runs on every PR to main (`.github/workflows/taugrid-validation.yml`). It builds, formats, vets, lints, and tests each Go module independently. The Python SDK has its own workflow triggered on `sdk/python/**` changes. Helm charts validate on `charts/**` changes against Helm 3 and 4.

Key CI settings: `GOFLAGS=-mod=readonly`, `GOTOOLCHAIN=local`. Tests run with `-count=1` (no caching). The controller module uses `-race`.

## TauGrid Release Version Bump

TauGrid uses one release version for the `taugrid`, `taugrid-core`, and
`tau-core-controller` charts plus the `tau`, `taugrid-portal`, and
`tau-core-controller` images. Helm and MCR use `X.Y.Z`; the Git tag uses
`vX.Y.Z`. The tag publishes the `tau` and `tau-gen` CLI binaries plus the
independently versioned Python SDK wheel; `gpu-monitoring` and `adx-mon` keep
their own release versions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/taugrid](https://github.com/Azure/taugrid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
