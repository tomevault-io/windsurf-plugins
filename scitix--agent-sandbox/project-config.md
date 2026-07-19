---
trigger: always_on
description: This file provides guidance and conventions for Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance and conventions for Claude Code when working with this repository.

---

## Project Overview

Agent Sandbox is a **Kubernetes Operator** that manages AI Agent sandbox Pod lifecycles. The core design uses a pre-warmed Pod pool combined with in-place image upgrades, avoiding the overhead of frequent Pod creation and deletion.

- **Go module**: `github.com/scitix/agent-sandbox`
- **Go version**: 1.25
- **CRD Group**: `agents.navix.sh`

**Cluster topology**: A worker cluster runs the AgentBox core (Operator + ExtProc + Sandbox Pods); an optional master cluster runs `/dashboard` (see `dashboard/CLAUDE.md`).

### Three Binaries

Each `cmd/*/main.go` is a thin stub that calls `Run()` from its sibling
`cmd/*/app` package — closed-source forks should import these packages rather
than re-implement the bootstrap.

| Binary | Entry | Bootstrap package | Responsibilities |
|--------|-------|-------------------|-----------------|
| `cmd/sandbox` | Operator / API Server | `cmd/sandbox/app` | REST API (`:8080`) + Controller Manager, optional E2B-compatible API (`:8090`) + Prometheus metrics (`:8082`) |
| `cmd/envoyextproc` | Data-plane ExtProc | `cmd/envoyextproc/app` | Envoy ExternalProcessor gRPC (`:9002`) + internal control-plane gRPC (`:9003`) |
| `cmd/wsproxy` | WS reverse-proxy sidecar | `cmd/wsproxy/app` | Listens on `:9003`, routes `/ws/clusters/{cid}/sandboxes/{id}/terminal` to the AgentBox API WebSocket endpoint; optional sync manager on `:9004` |

---

## Common Commands

```bash
# Code generation
make manifests          # Regenerate CRD YAML + RBAC (required after modifying api/v1alpha1/)
make generate           # Regenerate DeepCopy methods
make gen-all-api        # openapi.yaml → Go + TS (dashboard) + Python SDK (all in one)
make sync-crds-to-helm  # Sync CRDs + manager ClusterRole into Helm charts (run after make manifests)

# Build
make build              # Build all binaries (VERSION injected via -ldflags automatically)
make build-controller   # sandbox linux/amd64
make build-extproc      # envoyextproc linux/amd64
make build-wsproxy      # wsproxy binary
make lint-fix           # Run linter with auto-fix

# Version
make sync-version       # Sync VERSION file to openapi.yaml / pyproject.toml / __init__.py

# Testing
make test               # Unit tests (envtest, no cluster needed); packages run concurrently
make test-e2e           # E2E tests (requires a real cluster)
go test -tags=e2e ./test/e2e/ -v -ginkgo.v --ginkgo.focus="xxx"  # Run a single E2E test
```

> `make test` / `make build` automatically run `manifests → generate → gen-all-api → fmt → vet` first.

> **Before completion**: always run `make lint-fix` and confirm **0 issues**. The pre-push CI gate runs the same linter and will reject commits that contain lint errors.

---

## Change Checklist

1. Changed a default port in code → update the corresponding Service definition in Helm charts
2. Changed `api/v1alpha1/` CRD types → `make manifests generate` then `make sync-crds-to-helm`
3. Changed `pkg/openapi/native/openapi.yaml` → `make gen-all-api` (syncs Go + TS + Python SDK)
4. **Added/modified an API response field** → follow the full "API Field Addition SOP" below
5. Releasing a new version → see the "Version Management" section below

### API Field Addition SOP

The native API surface uses the generated `pkg/apiserver/gen` types directly as the single
externally-visible model. The service layer reads CRDs, projects them straight into
`gen.*` shapes, and the handler returns those untouched. There is no separate "domain"
mirror of wire fields, so the chain is shorter than it used to be.

Using Sandbox as an example, the complete change path for adding a field:

```
openapi.yaml                     ← 1. Define field schema
    ↓ make gen-all-api
gen/agentbox.gen.go              ← 2. Auto-generated (do not edit manually)
controllers/.../sandbox_pod.go   ← 3. Populate field in SandboxBaseFromPod / CaptureSandboxStopRecord
service/sandbox_service.go       ← 4. Ensure the field survives Create/Get/List/Delete paths
e2bcompat/domain/convert.go      ← 5. (Optional) map into the E2B compatibility shape
```

**Step-by-step checklist** (skip step 5 only when E2B compatibility isn't needed):

| # | File | Action | Verify |
|---|------|--------|--------|
| 1 | `pkg/openapi/native/openapi.yaml` | Add field definition to schema | `make gen-all-api` succeeds |
| 2 | `pkg/apiserver/gen/agentbox.gen.go` | Auto-generated; confirm new field exists | grep new field name |
| 3 | `pkg/controllers/sandboxpool/sandbox_pod.go` | Set the field on the `gen.Sandbox` returned by `SandboxBaseFromPod()` / `CaptureSandboxStopRecord()` | unit test |
| 4 | `pkg/apiserver/service/sandbox_service.go` | Make sure live (Create/Get/List) and historical (store) paths populate the field | unit test + API test |
| 5 | `pkg/e2bcompat/domain/convert.go` | If E2B needed, read from the `gen.Sandbox` argument and map into the E2B shape | E2B API test |

> **SandboxPool / SandboxTemplate fields follow the same pattern**: edit `poolToGen()` /
> `templateFromCRD()` in `pkg/apiserver/service/` (they are the only place the projection
> happens; the handler simply forwards the gen value).

### Where types live


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scitix/Agent-Sandbox](https://github.com/scitix/Agent-Sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
