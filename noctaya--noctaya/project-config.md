---
trigger: always_on
description: Noctaya owns the Kubernetes orchestration and lifecycle layer for scale-to-zero LLM serving:
---

# Noctaya Agent Guide

## Scope and sources of truth

Noctaya owns the Kubernetes orchestration and lifecycle layer for scale-to-zero LLM serving:
declarative workloads, runtime selection, model loading and caching, health, accelerator scheduling translation, autoscaling integration, and stable metrics.

Do not move inference kernels, vendor runtime behavior, device plugins, schedulers, monitoring lifecycle, fleet routing, or datacenter serving into Noctaya. Vendor adapters must remain thin Kubernetes translations.

The API group is `serving.noctaya.io/v1alpha1`:

- `LLMService` is namespaced and describes serving intent.
- `InferenceRuntime` is cluster-scoped and describes a reusable runtime and accelerator profile.
Its controller is passive; the `LLMService` reconciler consumes it.

Read the relevant source before editing:

- `docs/architecture.md` defines the project boundary and lifecycle.
- `CONTRIBUTING.md` defines the human workflow.
- API types and generated CRDs define schemas.
- The Makefile and `.github/workflows/` define commands and CI.
- Physical-validation reports define hardware claims.

Keep tutorials in the appropriate document under `docs/`, not in this guide.

## Repository map

| Path | Responsibility |
|---|---|
| `cmd/` | Operator and gateway entry points |
| `api/v1alpha1/` | CRD types and Kubebuilder markers |
| `internal/controller/llmservice/` | Serving lifecycle reconciler and envtest coverage |
| `internal/controller/inferenceruntime/` | Passive runtime controller |
| `internal/backend/runtime/` | Runtime contracts and shared vLLM rendering |
| `internal/backend/resources/` | Kubernetes resource builders and rendering tests |
| `internal/backend/{ascend,nvidia,registry}/` | Thin vendor adapters and built-in registration |
| `internal/gateway/{proxy,scaler,demand}/` | Admission, routing, demand reporting, and External Push scaling |
| `internal/model/` | Model URI resolution |
| `config/` and `charts/noctaya/` | Kustomize and manually maintained Helm packaging |
| `examples/` | Device profiles and optional integrations |
| `docs/validation/` | Shared validation requirements and physical-device reports |
| `docs/noctaya/` | Docusaurus presentation for `noctaya.io` |
| `test/e2e/` | Disposable Kind External Push lifecycle |
| `test/vllm-stub/` | CPU-only vLLM protocol stub |

There is one API group and no webhook. Do not move Kubebuilder-owned files or scaffold APIs or webhooks unless explicitly requested.
When scaffolding is required, use Kubebuilder and preserve every `+kubebuilder:scaffold:*` marker.

## Work safely

Inspect `git status` before editing. Existing changes belong to the user: preserve unrelated work, avoid overlapping broad rewrites, and never discard changes to clean the tree. Do not commit unless the user requests it.

Never hand-edit:

- `api/v1alpha1/zz_generated.deepcopy.go`;
- `config/crd/bases/*.yaml` or `config/rbac/role.yaml`;
- `charts/noctaya/crds/*.yaml`;
- `internal/gateway/externalscaler/*.pb.go`; or
- `PROJECT`.

After changing API types, validation/default markers, scope, or RBAC markers, run:

```bash
make manifests generate
make helm-crds
```

After changing `externalscaler.proto`, run:

```bash
go generate ./internal/gateway/externalscaler
```

Review generated diffs and reject unrelated churn. Helm templates are not generated from `config/`; align RBAC, manager settings, images, and CRDs across both installation paths. `make deploy` and `make build-installer` may change the manager image under `config/manager/`.

## Architecture invariants

One `LLMService` normally owns a backend Deployment and Service, gateway Deployment and public Service, optional cache and prewarm resources, a KEDA `ScaledObject`, and an internal ExternalScaler Service. Multiple gateways also use one aggregate-scaler Deployment.

Preserve these rules:

- Reconciliation is idempotent.
- Mutable owned resources use server-side apply with field owner `noctaya-operator` and controller references.
- Cache PVCs and prewarm Jobs are create-once because they contain immutable fields.
- KEDA is required for scaling but installed independently. Never bundle it with the chart or import its Go SDK; its CRD must exist before an `LLMService` is deployed.
- Monitoring remains outside reconciliation; optional resources belong in `examples/observability/`.
- Watch owned resources through controller-runtime instead of periodic requeues.
- Update status only when changed, use `metav1.Condition`, and set `ObservedGeneration`.
- Backend builders never set replicas; KEDA owns the backend `0..N` count.
- Gateway and backend replicas remain separate. Multiple gateways publish sequenced, expiring demand reports to the per-service aggregate scaler; do not bypass that path.
- Vendor behavior stays behind `internal/backend/runtime.BackendAdapter`; shared vLLM behavior stays common.
- Keep external CRDs unstructured unless a typed dependency is an explicit design decision.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noctaya/noctaya](https://github.com/noctaya/noctaya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
