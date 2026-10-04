---
trigger: always_on
description: SPDX-FileCopyrightText: 2026 INDUSTRIA DE DISEÑO TEXTIL, S.A. (INDITEX, S.A.)
---

<!--
SPDX-FileCopyrightText: 2026 INDUSTRIA DE DISEÑO TEXTIL, S.A. (INDITEX, S.A.)

SPDX-License-Identifier: Apache-2.0
-->

# AGENTS.md

Reference guide for AI agents and automated tools working on the Redkey Robin codebase.

---

## Project Summary

**Redkey Robin** is the companion runtime used by Redkey Operator to process configuration rollouts for a single `Redkey`.

It runs as a standalone Go binary that:

- connects to the Kubernetes API,
- watches the `RedkeyConfig` custom resources produced by Redkey Operator,
- selects the next actionable config revision for one cluster,
- advances config lifecycle state sequentially through the reconciliation loop, and
- exposes Prometheus metrics for the process.

Today, Robin focuses on orchestration of `RedkeyConfig` progression rather than cluster deployment. It operates per cluster instance, using `--cluster-name` and `--namespace` to scope its work.

### Technology Stack

| Layer | Technology |
| ----- | ---------- |
| Language | Go 1.26.8 |
| Runtime style | Standalone controller-style daemon |
| Kubernetes client | [controller-runtime](https://sigs.k8s.io/controller-runtime) v0.24.0 |
| API dependency | `github.com/inditextech/redkey-operator/api/v1beta1` via local `replace ../redkey-operator` |
| Metrics | [Prometheus client_golang](https://github.com/prometheus/client_golang) |
| Testing | Go `testing`, [Ginkgo v2](https://github.com/onsi/ginkgo) + [Gomega](https://github.com/onsi/gomega), [envtest](https://sigs.k8s.io/controller-runtime/tools/setup-envtest) |
| Linting | [golangci-lint](https://github.com/golangci/golangci-lint) v2.1.0 |

---

## Repository Layout

Key paths to understand before changing code:

- `cmd/main.go`: process entrypoint, flag parsing, startup wiring.
- `internal/config/`: shared runtime configuration consumed by the process.
- `internal/health/`: health/readiness endpoints and related plumbing.
- `internal/kubernetes/`: Kubernetes client helpers and cluster interactions.
- `internal/metrics/`: Prometheus collection and export logic.
- `internal/reconciler/`: config selection, state transitions, and reconciliation loop.
- `internal/redis/`: Redis connectivity, INFO/CLUSTER parsing, and helpers.
- `test/integration/`: envtest-based integration coverage.

Operational assumptions:

- Robin is a single-cluster runtime: one process instance is scoped to one `Redkey`.
- The sibling checkout at `../redkey-operator` is part of the expected local layout and provides the CRD/API types used by this module.
- Integration tests load CRDs from `../redkey-operator/config/crd/bases`.

---

## Build Commands

All common operations are driven by `make`. Tools that are not yet present are downloaded automatically into `bin/`.

This repository does not use Maven. There is no `pom.xml` or `mvnw` in Robin, so agents must not suggest `mvn` commands here. If an external workflow or template expects Maven goals/phases, use the following equivalents.

### Maven goal equivalents

| Maven goal or phase | Robin command | Notes |
| ------------------- | ------------- | ----- |
| `mvn validate` | `make tidy && make fmt && make vet && make lint` | Closest pre-test validation sequence. |
| `mvn test` | `make test` | Unit tests only. Generates `cover.out`. |
| `mvn failsafe:integration-test` | `make test-integration` | Runs envtest-based integration tests. |
| `mvn failsafe:verify` | `make test-integration` | Same integration suite; there is no separate Maven-style verify step. |
| `mvn verify` | `make lint && make test-all` | Preferred full local quality gate. |
| `mvn package` | `make build` | Produces `bin/robin`. |
| `mvn install` | not applicable | No Maven-style local artifact install phase exists. |

### Prerequisites (installed manually)

- Go 1.26+
- Make
- Docker or Podman (only needed for image build targets)
- Access to the sibling `../redkey-operator` checkout, because this module imports its API types through a local `replace`

### Dependency and formatting tasks

```shell
make tidy        # go mod tidy
make fmt         # go fmt ./...
make vet         # go vet ./...
make lint        # golangci-lint run
make lint-fix    # golangci-lint run --fix
make lint-config # validates golangci-lint configuration
```

### Binary build

```shell
make build       # go build -o bin/robin cmd/main.go
```

### Local execution

```shell
make run                                 # runs Robin locally
make run CLUSTER_NAME=mycluster NAMESPACE=mynamespace
```

Robin requires `--cluster-name` and `--namespace`. It also exposes `--metrics-bind-address`, `--reconcile-interval`, `--reconcile-interval-on-error`, and `--reconcile-interval-on-wait`.

### Container image

```shell
make docker-build              # builds image tagged as localhost:5005/redkey-robin:<VERSION>
make docker-push               # pushes the image
make docker-buildx             # cross-platform build (linux/amd64 + linux/arm64) and push
```

Override the image tag with `IMG=<registry>/<name>:<tag>`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [InditexTech/redkey-robin](https://github.com/InditexTech/redkey-robin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
