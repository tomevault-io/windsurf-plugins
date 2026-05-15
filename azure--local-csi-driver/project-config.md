---
trigger: always_on
description: <!-- markdownlint-disable MD013 -->
---

<!-- markdownlint-disable MD013 -->

# Copilot Instructions for local-csi-driver

## Project Overview

A Kubernetes CSI (Container Storage Interface) driver for Azure
local NVMe disks, built on
[controller-runtime](https://github.com/kubernetes-sigs/controller-runtime).
All controllers, webhooks, and manager setup must strictly follow
controller-runtime architecture and conventions. Two binaries:

- **`cmd/driver/main.go`** — DaemonSet CSI node/controller
  server. Manages LVM volumes, runs garbage collection
  controllers (PV failover, orphan cleanup), and emits startup
  diagnostic events.
- **`cmd/manager/main.go`** — Deployment with webhooks
  (validating + mutating) and PV cleanup controller. Uses
  leader election and cert rotation via OPA cert-controller.

No CRDs — operates on standard Kubernetes PersistentVolumes
and PersistentVolumeClaims.

## Build, Test, and Lint

```bash
make build                  # Build both driver and manager binaries
make build-driver           # Build driver binary only
make build-manager          # Build manager binary only
make test                   # Unit tests with race detection and coverage
make lint                   # golangci-lint v2
make lint-fix               # golangci-lint with auto-fix
make mocks                  # Regenerate mocks (go generate ./...)
pre-commit run --all-files  # Run all pre-commit hooks
```

### Pre-commit hooks

Pre-commit hooks are required. Install with `pip install pre-commit && pre-commit install`. They run automatically on `git commit` and include golangci-lint, hadolint (Dockerfiles), markdownlint, codespell, actionlint, shellcheck, copyright header injection, and Helm chart validation (kubeconform). Run manually with `pre-commit run --all-files`.

### Running a single test

```bash
# Single unit test by name
go test ./internal/csi/core/lvm/... -run TestStartupDiagnostic -v

# Single package
go test ./internal/gc/... -v

# E2E by pattern (Ginkgo focus)
FOCUS="should create volume" make e2e

# E2E by label
LABEL_FILTER="quick" make e2e
```

### Test types

| Command                    | What it tests                            |
| -------------------------- | ---------------------------------------- |
| `make test`                | Unit tests (excludes `test/` directory)  |
| `make test-e2e`            | E2E in Kind cluster                      |
| `make test-e2e-aks`        | E2E on AKS                               |
| `make test-sanity`         | CSI spec conformance (csi-test)          |
| `make test-scale SCALE=N`  | Load testing                             |

Integration tests (`suite_test.go` files) require `kubebuilder` binaries (etcd, kube-apiserver) via envtest — they will fail locally without these installed.

## Architecture

```text
cmd/
├── driver/main.go              # CSI server DaemonSet binary
└── manager/main.go             # Webhook + controller Deployment
internal/
├── csi/                        # CSI gRPC server implementation
│   ├── controller/             # CSI ControllerServer
│   ├── node/                   # CSI NodeServer
│   ├── core/                   # Volume operation interfaces
│   │   └── lvm/                # LVM implementation + diagnostics
│   ├── server/                 # Combined gRPC server
│   └── mounter/                # Mount/unmount abstraction
├── gc/                         # Garbage collection controllers
│   ├── failover_pv_cleanup.go  # PV node mismatch reconciler
│   └── lvm_orphan_cleanup.go   # Periodic orphan scanner
├── manager/pvcleanup/          # Released PV finalizer cleanup
├── webhook/
│   ├── enforceEphemeral/       # Validating webhook (PVCs)
│   └── hyperconverged/         # Mutating webhook (pod affinity)
└── pkg/                        # Shared libraries
    ├── events/                 # Event recorder wrappers
    ├── lvm/                    # LVM manager interface + mock
    ├── block/                  # Block device utils + mock
    ├── probe/                  # NVMe device discovery + mock
    └── telemetry/              # OTel tracing + Prometheus
```

## Key Conventions

### Events API (controller-runtime v0.23+)

This codebase uses the **new** Kubernetes events API
(`k8s.io/client-go/tools/events`), not the old
`k8s.io/client-go/tools/record` API.

```go
// ✅ Correct — new events API
import kevents "k8s.io/client-go/tools/events"

recorder kevents.EventRecorder
recorder.Eventf(
    obj, nil, corev1.EventTypeWarning,
    "Reason", "Reason", "message: %s", detail,
)

// ❌ Wrong — old API, do not use
import "k8s.io/client-go/tools/record"
recorder.Event(obj, corev1.EventTypeWarning, "Reason", "msg")
```

The local `internal/pkg/events` package wraps this with an
`ObjectRecorder` pattern for binding a recorder to a specific
object:

```go
// returns ObjectRecorder
recorder := events.WithObject(baseRecorder, obj)
recorder.Event(corev1.EventTypeNormal, "Reason", "message")
```

Event recording is feature-flagged. Both binaries create a
no-op recorder by default:

```go
recorder := events.NewNoopRecorder()
if eventRecorderEnabled {
    recorder = mgr.GetEventRecorder("local-csi-driver")
}
```

In tests, use `kevents.NewFakeRecorder(bufSize)`
(not `record.NewFakeRecorder`).

### Import aliases (enforced by linter)

```go
corev1    "k8s.io/api/core/v1"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/local-csi-driver](https://github.com/Azure/local-csi-driver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
