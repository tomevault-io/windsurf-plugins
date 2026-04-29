---
trigger: always_on
description: This document provides guidance for working with code in this repository.
---

# Kausality Development Guidelines

This document provides guidance for working with code in this repository.

## Build and Test Commands

```bash
make build          # Build binaries to bin/
make test           # Run unit tests with coverage
make envtest        # Run envtest integration tests (real API server)
make lint           # Run golangci-lint
make lint-fix       # Run golangci-lint with auto-fix
make gen            # Generate CRD manifests and DeepCopy methods

# Run a single test
go test ./pkg/drift -run TestIsControllerByHash -v

# Run envtests only
go test ./pkg/admission -tags=envtest -run TestDriftDetection -v
```

## Architecture

Kausality is an admission-based drift detection system for Kubernetes. It detects when controllers make unexpected changes to child resources.

### Components

| Component | Binary | ServiceAccount | Purpose |
|-----------|--------|----------------|---------|
| **Webhook** | `kausality-webhook` | `kausality-webhook` | Intercepts mutations and detects drift |
| **Controller** | `kausality-controller` | `kausality-controller` | Watches Kausality CRDs, reconciles webhook config |

**RBAC:**

| ClusterRole | Bound To | Purpose |
|-------------|----------|---------|
| `kausality-webhook` | webhook | Read policies and namespaces |
| `kausality-webhook-resources` | webhook | Aggregated access to tracked resources |
| `kausality-controller` | controller | Manage CRDs, webhook config, per-policy ClusterRoles |

### Core Concept: Drift Detection

**Drift** = controller making changes when parent hasn't changed (`generation == observedGeneration`)

| Actor | Parent State | Result |
|-------|--------------|--------|
| Controller | gen != obsGen | Expected (reconciling) |
| Controller | gen == obsGen | **Drift** |
| Different actor | any | New causal origin (not drift) |

### Controller Identification via User Hash Tracking

The system identifies controllers by tracking which users update parent status vs child spec. Users are identified by `UserInfo.Username` with fallback to `UserInfo.UID` for users without names:

**Annotations:**
- Parent: `kausality.io/controllers` - 5-char base36 hashes of user identifiers who update status (max 5)
- Child: `kausality.io/updaters` - 5-char base36 hashes of user identifiers who update spec (max 5)

**Recording:**
- Child CREATE/UPDATE (spec change only): hash added to child's `updaters` annotation (sync, via patch)
- Parent status UPDATE: hash added to parent's `controllers` annotation (sync, via direct API call), plus `kausality.io/observedGeneration` set to current generation (synthetic observedGeneration for parents without native `status.observedGeneration`)

Metadata-only changes do NOT record updaters - only spec changes do.

Note: Controllers and observedGeneration annotations use direct API calls because status subresource patches to metadata are ignored by Kubernetes.

**Detection logic:**
```
if no controller ownerRef → skip (can't be drift)

if child has 1 updater:
    controller = that single updater
else if parent has controllers annotation:
    controller = intersection(child.updaters, parent.controllers)
else:
    → can't determine, be lenient (skip drift detection)

if current_user_hash in controller set → controller request → check drift
else → not controller → not drift (new causal origin)
```

**Webhook configuration:** Must intercept status subresource updates to record controllers.

For detailed design documentation, see [doc/design/INDEX.md](doc/design/INDEX.md).

### Package Structure

- **`pkg/controller/`** - Controller identification via user hash tracking
  - `tracker.go` - `UserIdentifier()`, `HashUsername()`, `RecordUpdater()`, `RecordControllerAsync()`

- **`pkg/drift/`** - Core drift detection logic
  - `detector.go` - Main `Detector` with `Detect()` using user hash tracking
  - `resolver.go` - Resolves parent via controller ownerRef, extracts `ParentState` including `Controllers` hashes
  - `lifecycle.go` - Detects phases: Initializing, Initialized, Deleting
  - `types.go` - `DriftResult`, `ParentState`, `ParentRef`

- **`pkg/trace/`** - Causal trace propagation
  - `propagator.go` - `Propagate()` decides origin vs extend based on user hash
  - `types.go` - `Trace`, `Hop` types with JSON serialization

- **`pkg/admission/`** - Admission webhook handler
  - `handler.go` - Wraps drift detector + trace propagator for admission requests
  - `handler_envtest_test.go` - Comprehensive envtests against real API server

- **`pkg/approval/`** - Approval/rejection annotation handling
  - `types.go` - `Approval`, `Rejection`, `ChildRef` types
  - `checker.go` - Checks approvals against child references
  - `pruner.go` - Prunes consumed and stale approvals

- **`pkg/config/`** - Configuration handling
  - `config.go` - Per-resource enforce mode configuration

- **`pkg/callback/`** - Drift notification webhook callbacks
  - `v1alpha1/types.go` - `DriftReport`, `DriftReportResponse`, `ObjectReference`, `RequestContext`
  - `sender.go` - HTTP client for sending DriftReports to webhook endpoints
  - `tracker.go` - ID tracking for deduplication

- **`pkg/backend/`** - Backend server implementations
  - `server.go` - HTTP server with in-memory drift store
  - `store.go` - Thread-safe drift report storage


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kausality-io/kausality](https://github.com/kausality-io/kausality) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
