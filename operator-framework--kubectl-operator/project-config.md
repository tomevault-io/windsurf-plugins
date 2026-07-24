---
trigger: always_on
description: This document provides guidance for AI agents working with the kubectl-operator repository.
---

# AGENTS.md

This document provides guidance for AI agents working with the kubectl-operator repository.

---

## Repository Overview

**kubectl-operator** is a kubectl plugin that functions as a package manager for OLM Operators. It provides a CLI interface for managing both OLMv0 operators (legacy, through [Subscriptions](https://github.com/operator-framework/api/blob/master/crds/operators.coreos.com_subscriptions.yaml)) and OLMv1 extensions (next-generation, through ClusterExtensions).

---

## Architecture

### Directory Structure

```
kubectl-operator/
├── main.go                    # Entry point - delegates to internal/cmd
├── internal/
│   ├── cmd/                   # Command definitions
│   │   ├── root.go            # Root command with global flags
│   │   ├── catalog.go         # OLMv0 catalog commands
│   │   ├── operator.go        # OLMv0 operator commands
│   │   ├── olmv1.go           # OLMv1 root command, references the internal/olmv1 for command definitions.
│   │   └── internal/
│   │       ├── log/           # Logging utilities
│   │       └── olmv1/         # OLMv1 commands
│   ├── pkg/                   # Internal business logic packages
│   │   ├── action/            # OLMv0 command action implementations
│   │   ├── v1/
│   │   │   ├── action/        # OLMv1 command action implementations
│   │   │   └── client/        # Port-forwarding catalog client for OLMv1 search command
│   │   ├── operator/          # PackageManifest wrappers
│   │   ├── subscription/      # Subscription wrappers
│   │   ├── catalogsource/     # CatalogSource wrappers
│   │   └── operand/           # Operand strategy logic
│   └── version/               # Version information
├── pkg/
│   └── action/                # Configuration and client setup
├── .github/workflows/         # CI/CD pipelines
├── .bingo/                    # Bingo-managed tool dependencies
└── vendor/                    # Vendored dependencies
```

### Command Structure

```
kubectl operator
├── [OLMv0: Global Flags: --namespace, --timeout]
│
├── catalog                    # OLMv0 catalog management
│   ├── add                    # OLMv1: Add a catalogsource
│   ├── list                   # OLMv1: List catalogsources
│   └── remove                 # OLMv1: Remove a catalogsource
│
├── install                    # OLMv0: Install operator
├── upgrade                    # OLMv0: Upgrade operator
├── uninstall                  # OLMv0: Uninstall operator
├── list                       # OLMv0: List installed operators
├── list-available             # OLMv0: List available operators
├── list-operands              # OLMv0: List operands
├── describe                   # OLMv0: Describe operator
│
├── olmv1                      # OLMv1 resource management
│   ├── get
│   │   ├── extension         # OLMv1: Display clustrerextensions
│   │   └── catalog           # OLMv1: Display clustercatalogs
│   ├── create
│   │   └── catalog           # OLMv1: Create a clustercatalog
│   ├── delete
│   │   ├── extension         # OLMv1: Delete clustrerextensions
│   │   └── catalog           # OLMv1: Delete clustercatalogs
│   ├── update
│   │   ├── extension         # OLMv1: Update mutable fields on a clustrerextension
│   │   └── catalog           # OLMv1: Update mutable fields on a clustercatalog
│   ├── install
│   │   └── extension         # OLMv1: Create a clusterextension to install a package from a catalog
│   └── search
│       └── catalog           # OLMv1: Search packages in catalogs
│
└── version                    # Show version info
```

**Command Implementation Pattern:**
1. Command file in `internal/cmd/` creates Cobra command
2. Flags bound to action struct fields
3. Action instance created from `internal/pkg/action/` or `internal/pkg/v1/action/`
4. `Run(ctx context.Context)` method executes business logic
5. Results formatted and returned to user

### OLM Resources Reference

This section describes the Kubernetes resources that kubectl-operator interacts with. Note that kubectl-operator **does not define** these CRDs - they are defined by OLM itself.

#### OLMv0 Resources

| Resource | API Group | Purpose | kubectl-operator Interaction |
|----------|-----------|---------|------------------------------|
| **ClusterServiceVersion (CSV)** | operators.coreos.com/v1alpha1 | Defines operator metadata, installation strategy, permissions, and owned/required APIs | Created during `install`, monitored during installation, queried by `list`, deleted during `uninstall` |
| **Subscription** | operators.coreos.com/v1alpha1 | Tracks operator updates from a catalog channel; drives automatic upgrades | Created during `install`, updated during `upgrade`, queried by `list`, deleted during `uninstall` |
| **InstallPlan** | operators.coreos.com/v1alpha1 | Calculated list of resources to install/upgrade; requires approval | Created by OLM when Subscription is created, approved during `install` if auto-approval enabled |
| **CatalogSource** | operators.coreos.com/v1alpha1 | Repository of operators and metadata served via gRPC | Created by `catalog add`, listed by `catalog list`, deleted by `catalog remove` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [operator-framework/kubectl-operator](https://github.com/operator-framework/kubectl-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
