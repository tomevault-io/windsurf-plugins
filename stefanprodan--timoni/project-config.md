---
trigger: always_on
description: High-signal onboarding for AI coding agents working in Timoni.
---

# AGENTS.md

High-signal onboarding for AI coding agents working in Timoni.

## What Timoni is

Timoni is a package manager for Kubernetes powered by [CUE](https://cuelang.org/), inspired by Helm.
Apps are defined as **Modules** (type-safe CUE templates + config schema, distributed as OCI artifacts),
deployed as **Instances** (a module rendered with values and applied to a cluster), and composed via
**Bundles** (multiple instances declared together). Helm analogy: module ≈ chart, bundle ≈ umbrella chart,
instance ≈ release. A **Runtime** is the environment a bundle is applied into: it defines the target clusters
and the values fetched live from cluster resources (Secrets/ConfigMaps) for injection into bundles.

## Common commands

```shell
make build              # Build ./bin/timoni (CGO_ENABLED=0)
make test               # tidy + generate + fmt + vet + run all Go tests with Kubernetes envtest
make generate           # Regenerate api/v1alpha1/zz_generated.deepcopy.go via controller-gen
make cue-vet            # cue fmt + cue vet schemas, and `timoni mod vet` the example/blueprint/testdata modules
make docgen            # Regenerate docs/cmd/*.mdx from the cobra commands via `timoni docgen`
```

Run a single test:

```shell
go test ./cmd/timoni/... -run TestApply -v
```

- `make test` is the canonical pre-commit gate; it runs `go mod tidy`, code generation, fmt, vet and golangci-lint before testing, so a clean `make test` implies all of those pass.
- Tests in `cmd/timoni` and packages that talk to the API server require **envtest** (`make install-envtest` downloads the Kubernetes test binaries into `./bin`).
- After changing CUE schemas or example modules, run `make cue-vet`. The blueprint, example and testdata modules vendor the canonical `schemas/timoni.sh/core/v1alpha1` package as relative symlinks under `cue.mod/pkg`, so schema changes propagate without a sync step. After changing types in `api/v1alpha1`, run `make generate`.

## Architecture

The data flow for an `apply` (install/upgrade) is: **fetch module → build CUE → render K8s objects → reconcile onto cluster → record inventory**. Trace it through these layers:

### `cmd/timoni/` — CLI (cobra)
One file per command (e.g. `apply.go`, `build.go`, `bundle_apply.go`, `mod_push.go`), each with a sibling `_test.go`. `main.go` defines the root command, global flags (`--timeout`, `--namespace` via `kubeconfigArgs`, registry flags), and logger injection. Commands wire together the engine, reconciler, oci, and runtime packages — they hold little logic themselves. Tests inject a logger and drive the real cobra command, asserting with the gomega matchers and in-process OCI registry from `internal/testutils`. The CRD vendoring suite is the exception that uses on-disk golden fixtures — regen with the `timoni mod vendor crd …` commands documented at the top of `mod_vendor_crd_test.go`.

### `internal/engine/` — CUE acquisition and compilation
The core of Timoni. Turns CUE into Kubernetes objects:
- `ModuleBuilder` — compiles a module's CUE package into K8s objects, injecting instance name, namespace, module version, and `kubeVersion` (overridable via `TIMONI_KUBE_VERSION`). Default Kubernetes version is the `defaultKubeVersion` constant in `module_builder.go`.
- `ValuesBuilder` — merges `--values` overlays on top of the module's `values.cue`.
- `BundleBuilder` / `RuntimeBuilder` — compile Bundle and Runtime CUE definitions; bundles instantiate per-workspace with runtime values injected.
- `RuntimeInjector` — substitutes runtime values (cluster-read secrets/config) into bundles.
- `ResourceSet` — the rendered set of objects.
- `HealthCheck` — extracts the custom health checks a module declares under `timoni: healthChecks:` (the `#HealthCheck`/`#HealthCheckForCondition` CUE schemas) for custom resources that are not kstatus-compliant.
- `Importer` (`importer.go`) — generates CUE definitions from Kubernetes **CRDs** by converting their OpenAPI v3 schemas (this is what `timoni mod vendor crd` runs, letting module authors use custom resources type-safely).
- `CRDValidator` (`crd_validator.go`) — validates rendered custom resources with the kube-apiserver admission packages (OpenAPI schema, CEL rules, list uniqueness) against the original CRDs, which the `Importer` embeds in the generated `types_gen.cue` as a hidden `_crd` field and `timoni mod vet` collects from the module imports and output.
- `fetcher/` — pulls module sources, either `local.go` (filesystem path) or `oci.go` (OCI registry).

### `internal/reconciler/` — server-side apply
`Reconciler` (built via `NewReconciler`) takes the engine's build result and applies it using `github.com/fluxcd/pkg/ssa` (server-side apply): diff, apply, wait-for-ready, prune stale objects, handle force-recreate of immutable fields. `interactive.go` wraps it as `InteractiveReconciler` — the variant `apply`/`bundle apply` actually use — which adds the `--dry-run` (server-side dry-run) and `--diff` flows, rendering diffs via `internal/dyff`. Honors the `action.timoni.sh/*` annotations (`force`, `prune`, `one-off`, `wait`), all defined in `api/v1alpha1/actions.go`.

### `internal/runtime/` — cluster-side state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stefanprodan/timoni](https://github.com/stefanprodan/timoni) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
