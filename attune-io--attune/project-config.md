---
trigger: always_on
description: Attune: Kubernetes operator for in-place pod resource right-sizing (VPA replacement).
---

# AGENTS.md

## Project

Attune: Kubernetes operator for in-place pod resource right-sizing (VPA replacement).
Requires Kubernetes 1.32+ (In-Place Pod Resize; 1.32 alpha with feature gate, 1.33–1.34 beta enabled by default, 1.35+ GA). Built with Go 1.26,
controller-runtime v0.24.1, Kubebuilder v4, K8s API v0.36.1.

**Naming convention:** "Attune" (capitalized) in prose and documentation.
`attune` (lowercase) in code, packages, namespaces, Prometheus metrics
(`attune_resize_total`), CLI commands (`kubectl attune`), API groups
(`attune.io`), Helm chart names, Docker images, and URLs.

## Commands

- Install deps: `go mod download`
- Build: `make build`
- Build plugin: `make build-plugin`
- Build image: `make docker-build IMG=attune:dev`
- Test (unit): `make test`
- Test (single pkg): `go test ./internal/resize/... -race -count=1`
- Test (integration): `make test-integration`
- Test (E2E Chainsaw): `NO_COLOR=1 make test-e2e` (requires k3d cluster; NO_COLOR prevents raw ANSI codes in agent output)
- Test (E2E Go): `make test-e2e-go` (requires k3d cluster with operator + Prometheus)
- Test (E2E smoke): `make test-e2e-smoke` (requires deployed k3d/Kind cluster with operator + Prometheus)
- Test (fuzz): `make test-fuzz`
- Test (bench): `make test-bench`
- Lint: `make lint`
- Lint + fix: `make lint-fix`
- Format: `make fmt`
- Generate CRDs/RBAC: `make manifests`
- Generate deepcopy: `make generate`
- Helm chart docs: `make helm-docs-gen`
- Helm chart tests: `make helm-unittest`
- Helm lint + template validation: `make helm-lint`
- Doc defaults consistency check: `make verify-doc-defaults`
- Fast pre-commit checks: `make verify-quick` (no integration tests or govulncheck)
- All CI checks locally: `make verify`
- Clean build artifacts: `make clean`
- Local cluster (k3d): `make k3d-create && make k3d-deploy IMG=attune:e2e`
- Local cluster (Kind): `make kind-create && make kind-deploy IMG=attune:e2e`
- Full local test (auto-provisions k3d): `make test-local`
- Local smoke test (auto-provisions k3d): `make test-local-smoke`
- E2E tests: `make test-e2e` (requires local cluster with operator deployed)

## Structure

- `api/v1alpha1/` - CRD type definitions (AttunePolicy, AttuneDefaults)
- `cmd/manager/` - Operator entry point
- `cmd/kubectl-attune/` - kubectl plugin
- `internal/controller/` - Reconciler (core business logic)
- `internal/metrics/` - Metrics collection (Prometheus, Datadog, CloudWatch), QueryBuilder interface, rate limiting
- `internal/recommendation/` - Composable estimator chain (percentile, margin, confidence, bounds, change filter)
- `internal/resize/` - In-place pod resize engine via /resize subresource
- `internal/safety/` - Post-resize safety observation and rollback
- `internal/conflict/` - HPA conflict detection
- `internal/webhook/` - Admission webhooks (defaulting + validation)
- `internal/operatormetrics/` - Operator-level Prometheus metrics (init-registered)
- `internal/validation/` - Shared validation (Prometheus address SSRF checks)
- `internal/throttle/` - Shared throttle checker interface (breaks import cycle)
- `internal/transform/` - Informer cache transform functions (strip unused pod fields to reduce memory)
- `pkg/defaults/` - Shared default-value and merge logic (used by controller + kubectl plugin)
- `config/` - Kustomize manifests (CRDs, RBAC, manager deployment)
- `charts/attune/` - Helm chart with cert-manager webhook support
- `test/integration/` - envtest-based integration tests
- `test/e2e/` - Chainsaw E2E test scenarios
- `docs/` - MkDocs documentation site

## Conventions

### Import aliases (enforced by golangci-lint importas)

Use these exact aliases; the linter rejects alternatives:

```go
corev1      "k8s.io/api/core/v1"
appsv1      "k8s.io/api/apps/v1"
metav1      "k8s.io/apimachinery/pkg/apis/meta/v1"
apierrors   "k8s.io/apimachinery/pkg/api/errors"
ctrl        "sigs.k8s.io/controller-runtime"
```

### Logging

Use `logr` structured logging exclusively. `fmt.Print` and `fmt.Fprint` are
forbidden by the linter (except in `cmd/kubectl-attune/`).

### resource.Quantity

Use `resource.ParseQuantity()` (returns error) instead of `resource.MustParse()`
(panics). Use DecimalSI format for CPU, BinarySI for memory. Use Go `time.Duration`
for all durations (e.g., `168h` not `7d`).

### Float parsing

`strconv.ParseFloat("NaN", 64)` and `strconv.ParseFloat("Inf", 64)` succeed
with nil error, returning `math.NaN()` and `math.Inf()`. All float comparisons
with NaN return false, silently disabling any threshold or guardrail that uses
the parsed value. Always check `math.IsNaN(v) || math.IsInf(v, 0)` after
`strconv.ParseFloat` in validation code.

The same guard applies at **runtime query boundaries**, not just parse-time.
Prometheus queries, API responses, and external computations can return NaN
(e.g., `0/0` in PromQL) or Inf without any string parsing involved. Any
`float64` received from an external system must be checked before comparison.
Example: `internal/safety/monitor.go` SLO query values (PR #167),
`internal/metrics/collector.go` `GetThrottleRatio` (line 349).

### Webhooks

controller-runtime v0.24.x uses typed generic interfaces. Register webhooks with:

```go
// AttunePolicy: defaulting + validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [attune-io/attune](https://github.com/attune-io/attune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
