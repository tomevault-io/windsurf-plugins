---
trigger: always_on
description: Clowder is a Kubernetes operator that provisions and manages infrastructure for applications
---

# AGENTS.md

## Project Overview

Clowder is a Kubernetes operator that provisions and manages infrastructure for applications
running on the cloud.redhat.com platform. It defines three Custom Resource Definitions —
`ClowdApp`, `ClowdEnvironment`, and `ClowdJobInvocation` — and implements reconciliation
controllers that translate those CRDs into Deployments, Services, Secrets, and third-party
resources (Kafka topics, PostgreSQL instances, MinIO buckets, etc.). The same app definition
works unchanged across local development, testing, and production environments by swapping
provider implementations through `ClowdEnvironment` configuration.

## Dependencies

**Runtime:**
- Go 1.25 (minimum)
- Kubernetes cluster (Minikube for local development)
- Podman or Docker (container image builds)
- `kubectl` and `make`

**Key Go dependencies** (named only — no versions):
- `sigs.k8s.io/controller-runtime` (Kubebuilder reconciler framework)
- `k8s.io/api`, `k8s.io/client-go`, `k8s.io/apimachinery`
- `sigs.k8s.io/controller-tools` (`controller-gen` for CRD/RBAC generation)
- `github.com/onsi/ginkgo/v2` + `github.com/onsi/gomega` (unit tests)
- `kustomize` (manifest assembly)
- `gojsonschema` (JSON schema → Go config types)
- `github.com/RedHatInsights/rhc-osdk-utils` (resource cache)
- Strimzi, MinIO, cert-manager, KEDA, Caddy, PostgreSQL client libraries
- OpenTelemetry (OTLP), Prometheus client

**Test dependencies:**
- KUTTL (`kubectl kuttl`) — Kubernetes E2E tests
- `setup-envtest` — runs a real API server for unit tests
- Python 3 + pytest + `rh-clowder-ci` package (E2E test harness in `tests/rh_clowder_ci/`)

**Managed via:** `go.mod` (tools directive for build tooling); Renovate bot for automated updates.

## Development Commands

See [Development Setup][readme-dev] in the README for the full command reference.

Agent-relevant commands (code generation and validation):

```shell
# After modifying CRD types in apis/
make generate        # regenerate zz_generated.deepcopy.go
make manifests       # regenerate CRD YAML + RBAC manifests

# After modifying schema/schema.json
make genconfig       # regenerate controllers/cloud.redhat.com/config/types.go

# After modifying RBAC markers in controllers
make manifests

# Validate everything before committing
make pre-push        # runs fmt, vet, test, build, and regenerates docs
```

**CI (Tekton/Konflux):** `make -dn test` runs unit tests inside UBI9 go-toolset container.
**CI (GitHub Actions):** `golangci-lint` runs on pull requests; security scans run on push to master.

**Important:** Never modify generated files directly — they are overwritten by the above commands.
Files that should not be committed: `config/manager/kustomization.yaml`,
`controllers/cloud.redhat.com/version.txt` (reset with `make no-update`).

## Architecture

Clowder uses a provider plugin system where each service type (database, Kafka, object storage,
web, metrics, etc.) is implemented as a swappable provider registered by `init()`. Providers write
to an in-memory resource cache; the framework applies all changes atomically to Kubernetes at
reconciliation end. Entry point: `main.go`. Controllers: `controllers/cloud.redhat.com/`.
CRD types: `apis/cloud.redhat.com/v1alpha1/`. Config schema: `schema/schema.json`.

For full internal design, tradeoffs, the resource cache API, watch/filter system, code generation
pipeline, and dependency endpoint resolution, see [ARCHITECTURE.md][architecture].

## Code Style

- **Linter:** `golangci-lint` — configured in `.golangci.yml`. This is the active linter used in
  CI (`lint.yml` GitHub Action). No other linter config files are authoritative.
- **Formatters:** `gofmt` and `goimports` — both configured via `.golangci.yml` and enforced in CI.
- **Active linters:** `errcheck`, `gocritic`, `gosec`, `govet`, `ineffassign`, `revive`,
  `staticcheck`, `unused`, `bodyclose`.
- **Language version:** Go 1.25 minimum.
- **Python (test harness only):** `ruff` (line-length 100, E/F/W/I rules) in
  `tests/rh_clowder_ci/pyproject.toml`. Not enforced in CI.

## Common Mistakes

1. **Forgetting code generation after CRD or schema changes.** Modifying types in `apis/` without
   running `make generate && make manifests` leaves `zz_generated.deepcopy.go` and CRD manifests
   out of sync. Modifying `schema/schema.json` without running `make genconfig` leaves
   `controllers/cloud.redhat.com/config/types.go` stale. CI will fail.

2. **Writing to the Kubernetes API directly inside providers.** Providers must use the resource
   cache (`Cache.Create`, `Cache.Update`, `Cache.List`, `Cache.Get`) and must not call the API
   server directly. Resources are applied only at the end of reconciliation. Bypassing the cache
   can cause duplicate reconciliation triggers and race conditions.

3. **Wrong provider invocation order.** Providers that depend on resources created by earlier
   providers must have a higher integer order value in `ProvidersRegistration.Register(fn, order, name)`.
   If an earlier provider's resources are not yet in the cache when a later provider runs, the later
   provider will fail. Check existing order values in `provider.go` files before adding a new provider.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RedHatInsights/clowder](https://github.com/RedHatInsights/clowder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
