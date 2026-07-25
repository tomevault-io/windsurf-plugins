---
trigger: always_on
description: Manages the installation of resources defined in InstallPlans.
---

# AGENTS.md

This file provides AI agents with comprehensive context about the Operator Lifecycle Manager (OLM) v0 codebase to enable effective navigation, understanding, and contribution.

## Project Status

**CRITICAL**: This repository is in **maintenance mode**. OLM v0 accepts only critical bug fixes and security updates. For new development, use [operator-controller](https://github.com/operator-framework/operator-controller) (OLM v1).

## Project Overview

Operator Lifecycle Manager (OLM) extends Kubernetes to provide declarative installation, upgrade, and lifecycle management for Kubernetes operators. It's part of the [Operator Framework](https://github.com/operator-framework) ecosystem.

### Core Capabilities
- **Over-the-Air Updates**: Automatic operator updates via catalog channels
- **Dependency Resolution**: Automatic resolution and installation of operator dependencies
- **Multi-tenancy**: Namespace-scoped operator management via OperatorGroups
- **Discovery**: Catalog-based operator discovery and installation
- **Stability**: Prevents conflicting operators from owning the same APIs

## Architecture

OLM consists of two main operators working together:

### 1. OLM Operator (`cmd/olm`)
**Responsibility**: Manages the installation and lifecycle of operators defined by ClusterServiceVersions (CSVs)

**Key Functions**:
- Creates Deployments, ServiceAccounts, Roles, and RoleBindings from CSV specifications
- Manages CSV lifecycle states: None → Pending → InstallReady → Installing → Succeeded/Failed
- Monitors installed operator health and rotates certificates
- Enforces OperatorGroup namespace scoping

**Primary Controllers**:
- CSV Controller (pkg/controller/operators/olm)
- OperatorGroup Controller

### 2. Catalog Operator (`cmd/catalog`)
**Responsibility**: Manages operator catalogs, subscriptions, and dependency resolution

**Key Functions**:
- Monitors CatalogSources and builds operator catalogs
- Processes Subscriptions to track operator updates
- Generates InstallPlans with resolved dependencies
- Creates CRDs and CSVs from catalog content

**Primary Controllers**:
- Subscription Controller
- InstallPlan Controller
- CatalogSource Controller
- Registry Reconciler

## Custom Resource Definitions (CRDs)

| Resource | API Group | Owner | Description |
|----------|-----------|-------|-------------|
| **ClusterServiceVersion (CSV)** | operators.coreos.com/v1alpha1 | OLM | Defines operator metadata, installation strategy, permissions, and owned/required CRDs |
| **Subscription** | operators.coreos.com/v1alpha1 | Catalog | Tracks operator updates from a catalog channel; drives automatic upgrades |
| **InstallPlan** | operators.coreos.com/v1alpha1 | Catalog | Calculated list of resources to install/upgrade; requires approval (manual or automatic) |
| **CatalogSource** | operators.coreos.com/v1alpha1 | Catalog | Repository of operators and metadata; served via grpc from operator-registry |
| **OperatorGroup** | operators.coreos.com/v1 | OLM | Groups namespaces for operator installation scope; enables multi-tenancy |
| **OperatorCondition** | operators.coreos.com/v2 | OLM | Tracks operator health status and conditions |

## Directory Structure

```
operator-lifecycle-manager/
├── cmd/                          # Entry point binaries
│   ├── catalog/                  # Catalog Operator main
│   ├── olm/                      # OLM Operator main
│   ├── package-server/           # Package API server
│   └── copy-content/             # Content copy utility
│
├── pkg/                          # Core implementation
│   ├── api/                      # API client and wrappers
│   │   ├── client/               # Generated Kubernetes clients
│   │   └── wrappers/             # Client wrapper utilities
│   │
│   ├── controller/               # Main controllers
│   │   ├── bundle/               # Bundle lifecycle controller
│   │   ├── install/              # Installation controller
│   │   ├── operators/            # Operator/CSV controllers (OLM Operator)
│   │   └── registry/             # Catalog/registry controllers (Catalog Operator)
│   │
│   ├── lib/                      # Shared libraries and utilities
│   │   ├── catalogsource/        # CatalogSource utilities
│   │   ├── csv/                  # CSV manipulation utilities
│   │   ├── operatorclient/       # Operator client abstractions
│   │   ├── operatorlister/       # Informer-based listers
│   │   ├── operatorstatus/       # Status management
│   │   ├── ownerutil/            # Owner reference utilities
│   │   ├── queueinformer/        # Queue-based informers
│   │   ├── scoped/               # Scoped client for multi-tenancy
│   │   └── [other utilities]
│   │
│   ├── metrics/                  # Prometheus metrics
│   └── package-server/           # Package server implementation
│
├── test/                         # Testing infrastructure
│   ├── e2e/                      # End-to-end tests
│   └── images/                   # Test container images
│
├── doc/                          # Documentation
│   ├── design/                   # Architecture and design docs
│   └── contributors/             # Contributor guides
│
└── vendor/                       # Vendored dependencies
```

## Key Packages and Their Responsibilities


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [operator-framework/operator-lifecycle-manager](https://github.com/operator-framework/operator-lifecycle-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
