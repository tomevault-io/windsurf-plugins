---
trigger: always_on
description: The **Managed Upgrade Operator** is a Kubernetes operator written in Go that manages automated in-place cluster upgrades for OpenShift Dedicated Platform (OSD) and Azure Red Hat OpenShift (ARO). It orchestrates the upgrade process through pre-upgrade validation, capacity management, maintenance windows, and post-upgrade verification.
---

# Managed Upgrade Operator - Claude Code Development Guide

## Overview

The **Managed Upgrade Operator** is a Kubernetes operator written in Go that manages automated in-place cluster upgrades for OpenShift Dedicated Platform (OSD) and Azure Red Hat OpenShift (ARO). It orchestrates the upgrade process through pre-upgrade validation, capacity management, maintenance windows, and post-upgrade verification.

## Architecture & Technology Stack

### Core Technologies
- **Language**: Go 1.23.0 (with toolchain go1.23.9)
- **Framework**: Kubernetes Operator built with `operator-sdk v1.21.0`
- **Controller Framework**: `sigs.k8s.io/controller-runtime v0.20.0`
- **Testing**: Ginkgo v2 (BDD testing framework) + Gomega (matcher/assertion library)
- **Mocking**: GoMock for interface mocking
- **Container Base**: UBI9 minimal (registry.access.redhat.com/ubi9/ubi-minimal:9.6-1755695350)

### Key Dependencies
- **OpenShift APIs**: `github.com/openshift/api`, `github.com/openshift/client-go`
- **Cluster Version Operator**: `github.com/openshift/cluster-version-operator`
- **OCM SDK**: `github.com/openshift-online/ocm-sdk-go v0.1.494+` (official SDK for managed cluster integration)
  - Uses typed models from `clustersmgmt/v1` and `servicelogs/v1` packages
  - Replaces legacy custom OCM model structs
- **Prometheus**: Metrics collection and alerting management
- **Kubernetes APIs**: `k8s.io/api`, `k8s.io/client-go`, `k8s.io/apimachinery`

## Directory Structure

```
managed-upgrade-operator/
├── api/v1alpha1/              # Custom Resource Definitions (CRDs)
│   └── upgradeconfig_types.go # UpgradeConfig CRD definition
├── controllers/               # Kubernetes controllers
│   ├── upgradeconfig/         # Main upgrade orchestration controller
│   ├── nodekeeper/            # Node upgrade tracking controller
│   └── machineconfigpool/     # MachineConfigPool controller
├── pkg/                       # Core business logic packages
│   ├── upgraders/             # Cluster upgrader implementations (OSD, ARO)
│   ├── upgradesteps/          # Individual upgrade step implementations
│   ├── clusterversion/        # CVO interaction utilities
│   ├── drain/                 # Node draining strategies
│   ├── scaler/                # Node capacity scaling
│   ├── maintenance/           # Maintenance window management
│   ├── alertmanager/          # Alert silencing management
│   ├── notifier/              # Notification systems
│   ├── validation/            # Pre/post upgrade validations
│   ├── scheduler/             # Upgrade scheduling logic
│   ├── metrics/               # Prometheus metrics
│   └── configmanager/         # Configuration management
├── test/                      # Test infrastructure
│   ├── deploy/                # Test deployment manifests
│   └── e2e/                   # End-to-end tests
├── deploy/                    # Production deployment manifests
├── docs/                      # Documentation
├── boilerplate/               # Shared build tooling
├── build/                     # Container build files
│   └── Dockerfile             # Multi-stage container build
└── hack/                      # Build and maintenance scripts
```

## Core Concepts

### UpgradeConfig Custom Resource
The operator is driven by `UpgradeConfig` CRs that define:
- Target OpenShift version (`spec.desired.version`)
- Upgrade channel (`spec.desired.channel`)
- Upgrade start time (`spec.upgradeAt`)
- Drain timeout (`spec.PDBForceDrainTimeout`)
- Capacity reservation needs (`spec.capacityReservation`)
- Upgrade type: OSD or ARO (`spec.type`)

### External Service Integration

**OCM API Integration**:
- Uses official `ocm-sdk-go` with typed models
- Supports proxy environments (`HTTP_PROXY`, `HTTPS_PROXY`, `NO_PROXY`)
- Automatic retry with exponential backoff (5 retries, 2s initial delay, 30% jitter)
- Enhanced timeouts for high-latency environments (30s connection, 10s TLS handshake)

**Client Implementations**:
- `pkg/ocm`: External OCM API client (api.openshift.com) with proxy support
- `pkg/ocmagent`: Local OCM Agent client (cluster service) without proxy
- `pkg/dvo`: Deployment Validation Operator client with proxy support
- `pkg/maintenance`: AlertManager client with proxy support
- `pkg/metrics`: Prometheus metrics client with proxy support

### Controllers Architecture
1. **UpgradeConfig Controller**: Main orchestrator that processes UpgradeConfig CRs
2. **NodeKeeper Controller**: Monitors and remediates stuck node upgrades
3. **MachineConfigPool Controller**: Tracks MCP upgrade progress

### Upgrade Process Flow
1. **Pre-upgrade**: Health checks, capacity reservation, maintenance windows
2. **Control Plane Upgrade**: Triggers CVO to upgrade control plane
3. **Worker Node Upgrade**: Manages worker node drain and upgrade
4. **Post-upgrade**: Cleanup, health validation, notifications

## Build Commands

### Essential Development Commands
```bash
# Install tool dependencies
make tools

# Build the operator binary
make go-build

# Run tests
make go-test


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/managed-upgrade-operator](https://github.com/openshift/managed-upgrade-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
