---
trigger: always_on
description: Go-based Kubernetes operator for managing Paperclip instances (the open-source AI agent orchestration platform), built with controller-runtime (kubebuilder). CRD API group is `paperclip.ai`, version `v1alpha1`.
---

# CLAUDE.md -- Paperclip Kubernetes Operator

## Project Overview

Go-based Kubernetes operator for managing Paperclip instances (the open-source AI agent orchestration platform), built with controller-runtime (kubebuilder). CRD API group is `paperclip.ai`, version `v1alpha1`.

- **Module:** `github.com/paperclipinc/paperclip-operator`
- **Go version:** 1.25
- **GitHub:** `paperclipinc/paperclip-operator` (GHCR org: `paperclipinc`)

## Commands

```bash
make test              # Unit + integration tests (requires envtest binaries)
make lint              # golangci-lint
make build             # Build manager binary
make manifests         # Regenerate CRD YAML + RBAC after API type changes
make generate          # Regenerate deepcopy methods after API type changes
make sync-chart-crds   # Sync CRDs into Helm chart templates (run after make manifests)
make install           # Install CRDs into current cluster
make run               # Run operator locally against current cluster
make bench             # Run benchmarks for resource builders
make scorecard         # Run operator-sdk scorecard tests
make test-e2e          # Run E2E tests (requires Kind cluster)
go test ./internal/resources/ -v   # Fast unit tests (no envtest needed)
go vet ./...           # Go vet check
```

## Architecture

```
api/v1alpha1/          -> CRD types (Instance)
internal/controller/   -> Reconciliation logic (single controller + metrics)
internal/resources/    -> Pure resource builder functions (StatefulSet, Service, etc.)
config/crd/bases/      -> Generated CRD YAML (committed to git)
charts/                -> Helm chart (CRDs as templates in templates/crds/)
bundle/                -> OLM bundle for OperatorHub submissions
config/samples/        -> Example Instance CRs
hack/                  -> Build/sync scripts (sync-chart-crds.sh, check-helm-rbac-sync.sh)
.github/workflows/     -> CI/CD pipelines
```

**Separation of concerns:** Controller logic (`internal/controller/`) only orchestrates reconciliation. All resource construction happens in pure functions in `internal/resources/`. This makes builders easy to unit test without envtest.

## Paperclip-Specific Notes

- Paperclip is a Node.js app running on port 3100 (not a Go binary)
- Health endpoint: `GET /api/health`
- Requires `HOST=0.0.0.0` and `SERVE_UI=true` for Kubernetes
- WebSocket support needed in Ingress for real-time UI updates
- Heartbeat scheduler runs in the server process -- only one instance should run it in multi-replica setups
- Database modes: embedded (PGlite), external (connection string), managed (operator-provisioned PostgreSQL)
- Data directory: `/paperclip` (mounted as PVC)

## Reconciliation Rules

These rules are enforced by CI (Reconcile Guard check) and must be followed:

### Always use `controllerutil.CreateOrUpdate` for managed resources

Never call `r.Update()` or `r.Create()` directly on managed resources (StatefulSets, Services, ConfigMaps, etc.). Always use:

```go
obj := &appsv1.StatefulSet{
    ObjectMeta: metav1.ObjectMeta{
        Name:      resources.StatefulSetName(instance),
        Namespace: instance.Namespace,
    },
}
_, err := controllerutil.CreateOrUpdate(ctx, r.Client, obj, func() error {
    desired := resources.BuildStatefulSet(instance)
    obj.Labels = desired.Labels
    obj.Spec = desired.Spec
    return controllerutil.SetControllerReference(instance, obj, r.Scheme)
})
```

**Why:** Direct `r.Update()` calls are unconditional -- they update even when nothing changed, incrementing the resource generation, triggering a watch event, and causing an infinite reconciliation loop. `controllerutil.CreateOrUpdate` compares before/after and skips no-op updates.

**Exception:** `r.Update(ctx, instance)` on the CR itself is allowed for finalizer management. Add `// reconcile-guard:allow` for any other legitimate exceptions.

### Explicitly set all Kubernetes default values in builders

When building resources, always set fields that Kubernetes would default on the server side. If omitted, the desired spec differs from the stored spec on every reconcile.

### Preserve server-assigned fields

When updating resources, preserve fields assigned by the API server:
- Service: `ClusterIP`, `ClusterIPs`
- PVC: immutable after creation -- only create, never update

### Reconciliation return values

```go
return ctrl.Result{}, nil                    // Success, no requeue
return ctrl.Result{}, err                    // Requeue with exponential backoff
return ctrl.Result{Requeue: true}, nil       // Immediate requeue
return ctrl.Result{RequeueAfter: 5*time.Minute}, nil  // Requeue after delay
```

### Status management

- Use `meta.SetStatusCondition` for conditions (follows k8s API conventions)
- Track `ObservedGeneration` so consumers know the controller processed the latest spec
- Status subresource updates (`r.Status().Update`) must be separate from spec/metadata updates

### Owner references

Set `controllerutil.SetControllerReference` on all managed resources. This enables:
- Automatic garbage collection when the parent CR is deleted
- Automatic watch triggers when owned resources change


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paperclipinc/paperclip-operator](https://github.com/paperclipinc/paperclip-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
