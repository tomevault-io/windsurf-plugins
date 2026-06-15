---
trigger: always_on
description: Go-based Kubernetes operator for managing OpenClaw instances, built with controller-runtime (kubebuilder). CRD API group is `openclaw.rocks`, version `v1alpha1`.
---

# CLAUDE.md — OpenClaw Kubernetes Operator

## Project Overview

Go-based Kubernetes operator for managing OpenClaw instances, built with controller-runtime (kubebuilder). CRD API group is `openclaw.rocks`, version `v1alpha1`.

- **Module:** `github.com/paperclipinc/openclaw-operator`
- **Go version:** 1.25
- **GitHub:** `paperclipinc/openclaw-operator` (GHCR org: `paperclipinc`)

## Commands

```bash
make test          # Unit + integration tests (requires envtest binaries)
make lint          # golangci-lint
make build         # Build manager binary
make manifests     # Regenerate CRD YAML + RBAC after API type changes
make generate      # Regenerate deepcopy methods after API type changes
make install       # Install CRDs into current cluster
make run           # Run operator locally against current cluster
go test ./internal/resources/ -v   # Fast unit tests (no envtest needed)
go vet ./...       # Go vet check
```

## Architecture

```
api/v1alpha1/          → CRD types (OpenClawInstance)
internal/controller/   → Reconciliation logic (single controller)
internal/resources/    → Pure resource builder functions (Deployment, Service, etc.)
config/crd/bases/      → Generated CRD YAML (committed to git)
charts/                → Helm chart
bundle/                → OLM bundle for OperatorHub submissions
test/e2e/              → E2E tests (run against kind cluster)
```

**Separation of concerns:** Controller logic (`internal/controller/`) only orchestrates reconciliation. All resource construction happens in pure functions in `internal/resources/`. This makes builders easy to unit test without envtest.

## Reconciliation Rules

These rules are enforced by CI (Reconcile Guard check) and must be followed:

### Always use `controllerutil.CreateOrUpdate` for managed resources

Never call `r.Update()` or `r.Create()` directly on managed resources (Deployments, Services, ConfigMaps, etc.). Always use:

```go
obj := &appsv1.Deployment{
    ObjectMeta: metav1.ObjectMeta{
        Name:      resources.DeploymentName(instance),
        Namespace: instance.Namespace,
    },
}
_, err := controllerutil.CreateOrUpdate(ctx, r.Client, obj, func() error {
    desired := resources.BuildDeployment(instance)
    obj.Labels = desired.Labels
    obj.Spec = desired.Spec
    return controllerutil.SetControllerReference(instance, obj, r.Scheme)
})
```

**Why:** Direct `r.Update()` calls are unconditional — they update even when nothing changed, incrementing the resource generation, triggering a watch event, and causing an infinite reconciliation loop. `controllerutil.CreateOrUpdate` compares before/after and skips no-op updates.

**Exception:** `r.Update(ctx, instance)` on the CR itself is allowed for finalizer management. Add `// reconcile-guard:allow` for any other legitimate exceptions.

### Explicitly set all Kubernetes default values in builders

When building resources, always set fields that Kubernetes would default on the server side. If omitted, the desired spec differs from the stored spec on every reconcile.

**Deployment defaults to always set:**
- `RevisionHistoryLimit`, `ProgressDeadlineSeconds`
- `RestartPolicy`, `DNSPolicy`, `SchedulerName`, `TerminationGracePeriodSeconds`
- `TerminationMessagePath`, `TerminationMessagePolicy`, `ImagePullPolicy` on every container
- `SuccessThreshold` on every probe
- `DefaultMode` on ConfigMap volume sources

**Service defaults:** `SessionAffinity: None`

### Preserve server-assigned fields

When updating resources, preserve fields assigned by the API server:
- Service: `ClusterIP`, `ClusterIPs`
- PVC: immutable after creation — only create, never update

### Reconciliation return values

```go
return ctrl.Result{}, nil                    // Success, no requeue
return ctrl.Result{}, err                    // Requeue with exponential backoff
return ctrl.Result{Requeue: true}, nil       // Immediate requeue
return ctrl.Result{RequeueAfter: 5*time.Minute}, nil  // Requeue after delay
```

When returning `err != nil`, the `RequeueAfter` field is ignored (exponential backoff takes over).

### Status management

- Use `meta.SetStatusCondition` for conditions (follows k8s API conventions)
- Track `ObservedGeneration` so consumers know the controller processed the latest spec
- Status subresource updates (`r.Status().Update`) must be separate from spec/metadata updates

### Owner references

Set `controllerutil.SetControllerReference` on all managed resources. This enables:
- Automatic garbage collection when the parent CR is deleted
- Automatic watch triggers when owned resources change

## Coding Conventions

### Go style
- Use `0o644` (not `0644`) for octal literals - gocritic lint enforces this
- Wrap errors: `fmt.Errorf("context: %w", err)`
- Use the generic `Ptr[T]` helper from `internal/resources/common.go` for pointer values
- Never use em dashes or en dashes in code, comments, or strings - use regular hyphens/dashes (`-` or `--`) instead
- Run `make fmt` and `make lint` before committing

### Commit messages
Use conventional commits: `feat:`, `fix:`, `docs:`, `ci:`, `chore:`, `refactor:`, `test:`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paperclipinc/openclaw-operator](https://github.com/paperclipinc/openclaw-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
