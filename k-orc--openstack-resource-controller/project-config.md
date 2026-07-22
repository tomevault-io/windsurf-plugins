---
trigger: always_on
description: This document provides instructions for AI agents to develop controllers in the ORC project.
---

# OpenStack Resource Controller (ORC) - Development Guide

This document provides instructions for AI agents to develop controllers in the ORC project.

## Project Overview

ORC is a Kubernetes operator that manages OpenStack resources declaratively. Each OpenStack resource (Flavor, Server, Network, etc.) has a corresponding Kubernetes Custom Resource and controller.

**Key Principle**: ORC objects only reference other ORC objects, never OpenStack resources directly. OpenStack resource IDs appear only in status fields.

## Project Structure

```
openstack-resource-controller/
├── api/v1alpha1/              # CRD type definitions (*_types.go)
├── internal/
│   ├── controllers/           # Controller implementations
│   │   └── <resource>/        # Each controller in its own package
│   │       ├── controller.go  # Setup, dependencies, SetupWithManager
│   │       ├── actuator.go    # OpenStack CRUD operations
│   │       ├── status.go      # Status writer implementation
│   │       ├── zz_generated.*.go  # Generated code (DO NOT EDIT)
│   │       └── tests/         # KUTTL E2E tests
│   ├── logging/               # Log level constants
│   ├── osclients/             # OpenStack API client wrappers
│   ├── scope/                 # Cloud credentials & client factory
│   └── util/
│       ├── applyconfigs/      # SSA apply config helpers
│       ├── credentials/       # Credential watch & dependency setup
│       ├── dependency/        # Dependency framework
│       ├── errors/            # Error classification (Terminal, IsRetryable)
│       ├── finalizers/        # Finalizer helpers
│       ├── result/            # Result helpers
│       ├── strings/           # Finalizer/field-owner name generation
│       └── tags/              # Tag reconciliation utilities
├── cmd/
│   ├── manager/               # Main entry point
│   ├── models-schema/         # OpenAPI schema generation
│   ├── resource-generator/    # Code generation
│   └── scaffold-controller/   # New controller scaffolding
└── website/docs/development/  # Detailed documentation
```

## Architecture

### Generic Reconciler Framework

All controllers use a generic reconciler that handles the reconciliation loop. Controllers implement interfaces:

- **CreateResourceActuator**: Create and import operations
- **DeleteResourceActuator**: Delete operations
- **ReconcileResourceActuator**: Post-creation updates (optional)
- **ResourceStatusWriter**: Status and condition management

### Key Interfaces

Controllers implement these methods (see `internal/controllers/servergroup/` for a simple example):

```go
// Required by all actuators
GetResourceID(osResource) string
GetOSResourceByID(ctx, id) (*osResource, ReconcileStatus)
ListOSResourcesForAdoption(ctx, obj) (iterator, bool)

// For creation/import
ListOSResourcesForImport(ctx, obj, filter) (iterator, ReconcileStatus)
CreateResource(ctx, orcObject) (*osResource, ReconcileStatus)

// For deletion
DeleteResource(ctx, orcObject, osResource) ReconcileStatus

// Optional - for updates after creation
GetResourceReconcilers(ctx, obj, osResource, controller) ([]ResourceReconciler, ReconcileStatus)
```

### Two Critical Conditions

Every ORC object has these conditions:

1. **Progressing**
   - `True`: Spec doesn't match status; controller expects more reconciles
   - `False`: Either available OR terminal error (no more reconciles until spec changes)

2. **Available**
   - `True`: Resource is ready for use
   - Determined by `ResourceStatusWriter.ResourceAvailableStatus()`

### ReconcileStatus Pattern

Methods return `ReconcileStatus` instead of `error`:

`ReconcileStatus` is a type alias for a pointer (`type ReconcileStatus = *reconcileStatus`). `nil` is a valid value meaning "success, no reschedule", and all methods are safe to call on a nil receiver.

```go
nil                                          // Success, no reschedule
progress.WrapError(err)                      // Wrap error for handling
reconcileStatus.WithRequeue(5*time.Second)   // Schedule reconcile after delay
reconcileStatus.WithProgressMessage("...")   // Add progress message
progress.NeedsRefresh()                      // Immediate re-reconcile to refresh status after mutation
progress.WaitingOnOpenStack(progress.WaitingOnReady, 15*time.Second) // Poll for OpenStack state change
progress.WaitingOnObject("Network", name, progress.WaitingOnCreation) // Wait for a k8s object
reconcileStatus.WithReconcileStatus(other)   // Merge two ReconcileStatuses
```

### Error Classification

- **Transient errors** (5xx, API unavailable): Default handling with exponential backoff
- **Non-recoverable errors** (409 Conflict, non-HTTP gophercloud errors): Wrap with `orcerrors.Terminal()` - no retry

```go
// Non-recoverable error example
if err != nil {
    if !orcerrors.IsRetryable(err) {
        err = orcerrors.Terminal(orcv1alpha1.ConditionReasonInvalidConfiguration,
            "invalid configuration: "+err.Error(), err)
    }
    return nil, progress.WrapError(err)
}
```

## Dependencies

Dependencies are core to ORC - they ensure resources are created in order.

### Types of Dependencies

1. **Normal Dependency**: Wait for object to exist and be available

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [k-orc/openstack-resource-controller](https://github.com/k-orc/openstack-resource-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
