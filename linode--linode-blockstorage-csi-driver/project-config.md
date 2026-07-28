---
trigger: always_on
description: Handles node-local volume operations:
---

# Linode Block Storage CSI Driver - AI Coding Instructions

## Architecture Overview

This is a Kubernetes CSI (Container Storage Interface) driver for Linode Block Storage volumes. The driver implements the CSI specification with three main server components:

- **ControllerServer** (`internal/driver/controllerserver.go`) - Handles volume lifecycle (create/delete/attach/detach/resize)
- **NodeServer** (`internal/driver/nodeserver.go`) - Handles node-level operations (mount/unmount/stage/unstage)
- **IdentityServer** (`internal/driver/identityserver.go`) - Provides driver identity and capabilities

The driver runs in two modes:
- **Controller mode**: Manages volume lifecycle via Linode API
- **Node mode**: Handles local volume operations on Kubernetes nodes

## CSI RPC Implementation Patterns

### Controller Service (`ControllerServer`)
Implements volume lifecycle management with required capabilities:
- `CREATE_DELETE_VOLUME`: Volume provisioning/deprovisioning via Linode API
- `PUBLISH_UNPUBLISH_VOLUME`: Volume attachment/detachment to/from nodes
- `EXPAND_VOLUME`: Online/offline volume expansion support
- `CREATE_DELETE_SNAPSHOT`: Snapshot lifecycle management

### Node Service (`NodeServer`) 
Handles node-local volume operations:
- `STAGE_UNSTAGE_VOLUME`: Block device staging to global mount point
- `GET_VOLUME_STATS`: Volume usage statistics for monitoring
- `EXPAND_VOLUME`: Filesystem expansion after controller expansion

### Identity Service (`IdentityServer`)
Required by all CSI plugins:
- `GetPluginInfo`: Returns driver name (`linodebs.csi.linode.com`) and version
- `GetPluginCapabilities`: Advertises `CONTROLLER_SERVICE` and `VOLUME_ACCESSIBILITY_CONSTRAINTS`
- `Probe`: Health check with optional readiness reporting

### RPC Interaction Rules
Per CSI spec section on RPC interactions and reference counting:
- `NodeStageVolume` MUST be called before any `NodePublishVolume` 
- All `NodeUnpublishVolume` MUST complete before `NodeUnstageVolume`
- `ControllerUnpublishVolume` called after all Node operations complete
- CO responsible for reference counting when `STAGE_UNSTAGE_VOLUME` advertised

## Key Architectural Patterns

### Dependency Injection & Interface Design
The driver uses extensive dependency injection with interfaces for testability:
- `LinodeClient` interface wraps the Linode API client (`pkg/linode-client/`)
- `DeviceUtils` interface handles device discovery (`pkg/device-manager/`)
- `SafeMounter` interface wraps mount operations (`pkg/mount-manager/`)
- `Encryption` interface handles LUKS encryption (`internal/driver/luks.go`)

### Package Structure Convention
```
pkg/           # Reusable packages with interfaces
internal/      # Driver-specific logic
mocks/         # Generated mocks for testing
tests/         # E2E, CSI sanity, and upstream tests
```

### Error Handling
Use gRPC status codes throughout CSI implementations per spec requirements:
```go
// Volume doesn't exist
return status.Error(codes.NotFound, "volume not found")

// Volume already exists but incompatible  
return status.Error(codes.AlreadyExists, "volume exists with different parameters")

// Invalid parameters
return status.Error(codes.InvalidArgument, "missing required field: volume_id")

// Resource limits exceeded
return status.Error(codes.ResourceExhausted, "max volumes per node exceeded")
```

**Status Message Guidelines**: 
- MUST contain human-readable description if not `OK`
- MAY be surfaced to end users by CO
- SHOULD provide actionable information for debugging

## Critical Development Workflows

### Building & Testing
```bash
# Build in container (required for consistent cross-platform builds)
export DOCKERFILE=Dockerfile.dev  # Use dev Dockerfile for development
make docker-build

# Run unit tests (in container)
make test

# Generate mocks after interface changes
make generate-mock

# Full CI pipeline
make ci  # vet + lint + test + build
```

### E2E Testing Workflow
The project uses a sophisticated E2E setup with CAPI (Cluster API) and CAPL (Cluster API Provider Linode):

```bash
# Create management cluster + workload cluster with CSI driver
make mgmt-and-capl-cluster

# Run E2E tests with chainsaw
make e2e-test

# Run CSI sanity tests
make csi-sanity-test

# Clean up clusters
make cleanup-cluster
```

Environment variables required:
- `LINODE_TOKEN` - Linode API token
- `LINODE_REGION` - Target region for testing

### Release Process
```bash
make release IMAGE_VERSION=v1.2.3
```
Generates release manifests and Helm chart in `release/` directory.

## Project-Specific Conventions

### Volume Naming & Labeling
- Volume names use prefix pattern: `pvc-{uuid}` for K8s PVC integration
- Optional volume label prefix via `LINODE_VOLUME_LABEL_PREFIX` (max 12 chars, regex: `^[0-9A-Za-z_-]{0,12}$`)
- Volume tags support via CSI parameters (`volumeTags` annotation)
- Volume IDs are plugin-generated and MUST be unique within plugin scope

### Volume Size Constraints (CSI CapacityRange)
- Linode enforces 10Gi minimum - driver handles transparently by provisioning larger size
- `CapacityRange.required_bytes`: Minimum size (MUST be honored even if larger than Linode minimum)
- `CapacityRange.limit_bytes`: Maximum size (MUST NOT be exceeded)
- Driver MUST validate range and return `OUT_OF_RANGE` if unsupported

### Encryption (LUKS)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linode/linode-blockstorage-csi-driver](https://github.com/linode/linode-blockstorage-csi-driver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
