---
trigger: always_on
description: Cluster API Provider Azure (CAPZ) is a Kubernetes-native declarative infrastructure provider for managing Azure clusters. It implements the Cluster API (CAPI) specification for both self-managed (IaaS) and AKS-managed Kubernetes clusters on Azure.
---

## Overview

Cluster API Provider Azure (CAPZ) is a Kubernetes-native declarative infrastructure provider for managing Azure clusters. It implements the Cluster API (CAPI) specification for both self-managed (IaaS) and AKS-managed Kubernetes clusters on Azure.

**Key Concepts:**
- CAPI: Cluster API - the upstream Kubernetes project defining cluster lifecycle APIs
- CAPZ: Cluster API Provider Azure - this repository
- ASO: Azure Service Operator - used for declarative Azure resource management
- Reconciler: Controller-runtime pattern for managing Kubernetes custom resources

## Architecture

### Core Components

1. **API Definitions (`/api` and `/exp/api`)**
   - `v1beta1`: Stable API version for core resources
   - `v1alpha1`: Deprecated, use v1beta1
   - `/exp`: Experimental features (MachinePools, Managed clusters)
   - Key resources: AzureCluster, AzureMachine, AzureMachinePool, AzureManagedCluster, AzureManagedControlPlane

2. **Controllers (`/controllers` and `/exp/controllers`)**
   - Each controller reconciles a specific custom resource type
   - Controllers use Azure SDK and ASO to manage Azure infrastructure
   - Reconciliation pattern: observe state → determine actions → apply changes → update status
   - Key controllers: AzureClusterReconciler, AzureMachineReconciler, AzureMachinePoolReconciler

3. **Azure Services Layer (`/azure/services`)**
   - Service-specific Azure API clients organized by Azure resource type
   - Examples: `virtualnetworks`, `subnets`, `loadbalancers`, `virtualmachines`, `vmss`
   - Each service implements the `Reconciler` interface: `Reconcile()`, `Delete()`
   - Services handle Azure API calls, credential management, and error handling

4. **Scope Package (`/azure/scope`)**
   - Provides context and configuration for controllers
   - Scopes encapsulate cluster/machine specs, credentials, and Azure clients
   - Key scopes: ClusterScope, MachineScope, MachinePoolScope, ManagedControlPlaneScope

5. **Feature Gates (`/feature`)**
   - Controls experimental/optional functionality
   - Important gates: `MachinePool`, `ASOAPI`, `EdgeZone`, `ClusterResourceSet`

### Data Flow

```
User creates K8s resource → Controller watches → Reconciler triggered →
Scope created → Azure service methods called → Azure API interactions →
Status updated → Requeue if needed
```

### Two Deployment Models

1. **Self-Managed (IaaS)**: CAPZ creates VMs, networks, load balancers via Azure APIs
2. **Managed (AKS)**: CAPZ creates/manages AKS clusters and agent pools via ASO

## Development Commands

### Building and Testing

```bash
# Build the manager binary
make manager

# Run unit tests
make test

# Run unit tests with race detector
make test-cover

# Run linting
make lint

# Fix lint issues automatically
make lint-fix

# Generate code (deepcopy, CRDs, webhooks, mocks)
make generate

# Verify all generated files are up-to-date
make verify
```

### Running a Single Test

```bash
# Run specific test by name
KUBEBUILDER_ASSETS="$(make setup-envtest 2>&1 | grep -o '/.*')" go test -v -run TestFunctionName ./path/to/package

# Run all tests in a package
KUBEBUILDER_ASSETS="$(make setup-envtest 2>&1 | grep -o '/.*')" go test -v ./controllers/
```

### Docker Images

```bash
# Build controller image (defaults to dev tag)
make docker-build

# Build and push to registry
REGISTRY=myregistry.io make docker-build docker-push

# Build all architectures
make docker-build-all
```

### Local Development with Tilt

```bash
# Create Kind cluster and start Tilt
make kind-create tilt-up

# Use AKS as management cluster instead
make aks-create tilt-up

# Delete Kind cluster
make kind-reset
```

**tilt-settings.yaml** is required with Azure credentials (see docs/book/src/developers/development.md for details).

`make tilt-up` runs `check-az-cli`; if `az` is missing, it warns and tells you to install the Azure CLI and retry `make tilt-up` (needed for Tilt flows that call `az`, e.g. VNet peering with AKS as management cluster). Use `VERBOSE=1 make check-az-cli` to print the detected binary path.

### E2E Testing

```bash
# Run E2E tests (requires Azure credentials in env)
./scripts/ci-e2e.sh

# Run conformance tests
./scripts/ci-conformance.sh
```

## Important Patterns

### Controller Reconciliation

Controllers follow this pattern:
1. Fetch the resource being reconciled
2. Check deletion timestamp; run finalizer logic if deleting
3. Create/update scope with credentials and config
4. Call service Reconcile() or Delete() methods
5. Update resource status
6. Return result with requeue if needed

### Error Handling

- Transient errors: Return `ctrl.Result{RequeueAfter: timeout}` to retry
- Permanent errors: Log error, update status conditions, don't requeue
- Long-running operations: Use Azure async patterns with futures

### Adding New Azure Resources

1. Define API in `/api/v1beta1` or `/exp/api/v1beta1`
2. Run `make generate` to create deepcopy methods
3. Create controller in `/controllers`
4. Create service in `/azure/services/<resourcetype>`
5. Update scope if needed in `/azure/scope`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubernetes-sigs/cluster-api-provider-azure](https://github.com/kubernetes-sigs/cluster-api-provider-azure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
