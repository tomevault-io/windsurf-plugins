---
trigger: always_on
description: Dell Container Storage Modules (CSM) Operator is a Kubernetes operator built with Kubebuilder v4 and Go 1.26. It manages the lifecycle of Dell CSI drivers and CSM modules on Kubernetes clusters. The operator watches a custom resource (`ContainerStorageModule`, API group `storage.dell.com/v1`, namespaced) and reconciles the desired state by deploying, upgrading, or removing driver and module components.
---

# AGENTS.md -- Dell CSM Operator

## Project Overview

Dell Container Storage Modules (CSM) Operator is a Kubernetes operator built with Kubebuilder v4 and Go 1.26. It manages the lifecycle of Dell CSI drivers and CSM modules on Kubernetes clusters. The operator watches a custom resource (`ContainerStorageModule`, API group `storage.dell.com/v1`, namespaced) and reconciles the desired state by deploying, upgrading, or removing driver and module components.

Supported CSI drivers: PowerFlex, PowerMax, PowerScale, PowerStore, Unity, COSI.
Supported modules: Authorization (v1 and v2), Observability, Replication, Resiliency, Reverse Proxy.

Module path: `eos2git.cec.lab.emc.com/CSM/csm-operator`

Key dependencies: controller-runtime v0.22.4, k8s.io/api v0.35.0, cert-manager v1.19.3, go.uber.org/zap (structured logging), stretchr/testify (test assertions), Ginkgo v2 (E2E tests).

## Architecture

### Reconciliation Loop

The core logic lives in `controllers/csm_controller.go` (ContainerStorageModuleReconciler). On each reconciliation:

1. The reconciler reads the ContainerStorageModule CR.
2. It selects the correct driver or module configuration from YAML templates in `operatorconfig/`.
3. It performs template substitution (image names, versions, feature flags) to produce Kubernetes manifests.
4. It applies those manifests through typed resource handlers in `pkg/resources/` (one handler per resource kind: Deployment, DaemonSet, ConfigMap, CSIDriver, RBAC, ServiceAccount).
5. It updates the CR status to reflect the current state.

### Driver and Module Implementations

Each driver has a dedicated file in `pkg/drivers/` (e.g., `powerflex.go`, `powermax.go`). Each module has a dedicated file in `pkg/modules/` (e.g., `authorization.go`, `observability.go`). These files contain the logic to parse the CR spec, validate configuration, and produce the set of Kubernetes objects to reconcile.

Common configuration logic shared across drivers or modules lives in `commonconfig.go` in the respective package.

### Operator Config Templates

The `operatorconfig/` directory contains YAML templates that define the Kubernetes resources for each driver and module version:

- `operatorconfig/driverconfig/{driver}/{version}/` -- Per-driver versioned configs. Each version directory contains `controller.yaml`, `node.yaml`, `csidriver.yaml`, `driver-config-params.yaml`, and `upgrade-path.yaml`.
- `operatorconfig/driverconfig/common/` -- Shared driver configuration (e.g., `default.yaml`).
- `operatorconfig/moduleconfig/{module}/` -- Per-module configurations for authorization, observability, replication, resiliency, and csireverseproxy.

When adding a new driver or module version, create a new version directory under the appropriate path and update the upgrade path file.

### Resource Handlers

`pkg/resources/` contains sub-packages for each Kubernetes resource type the operator manages: `deployment/`, `daemonset/`, `configmap/`, `csidriver/`, `rbac/`, `serviceaccount/`. These implement creation, update, and deletion logic and are called by the reconciler.

### Utilities

`pkg/operatorutils/` provides core utility functions (`utils.go`, `common.go`, `status.go`) for manifest parsing, version selection, status management, and common operations used throughout the codebase.

## Build Commands

All commands use the top-level Makefile.

| Command | Description |
|---|---|
| `make build` | Build the manager binary (with `vendor`, outputs `bin/manager`) |
| `make test` | Run all tests with coverage (generates CRDs first) |
| `make unit-test` | Run unit tests with 90% coverage threshold |
| `make controller-unit-test` | Unit tests for `controllers/` only |
| `make driver-unit-test` | Unit tests for `pkg/drivers/` only |
| `make module-unit-test` | Unit tests for `pkg/modules/` only |
| `make operatorutils-unit-test` | Unit tests for `pkg/operatorutils/` only |
| `make manifests` | Generate CRDs and RBAC manifests (controller-gen v0.18.0) |
| `make generate` | Generate DeepCopy/DeepCopyInto/DeepCopyObject methods |
| `make fmt` | Run `go fmt ./...` |
| `make vet` | Run `go vet ./...` |
| `make lint` | Run golangci-lint (builds first) |
| `make run` | Run the controller locally against the current kubeconfig |
| `make deploy` | Deploy the operator to the cluster |
| `make undeploy` | Remove the operator from the cluster |
| `make install` | Install CRDs into the cluster |
| `make uninstall` | Remove CRDs from the cluster |
| `make bundle` | Generate OLM bundle manifests |
| `make images` | Build the container image |
| `make static-manifests` | Generate static CRD and operator YAML in `deploy/` |
| `make vendor` | Run `go mod vendor` (requires GOPRIVATE for internal modules) |
| `make gen-semver` | Regenerate semantic version files in `core/` |
| `make tidy` | Run `go mod tidy` for both the main module and `tests/e2e/` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dell/csm-operator](https://github.com/dell/csm-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
