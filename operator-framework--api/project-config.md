---
trigger: always_on
description: This file provides AI agents with comprehensive context about the operator-framework/api repository to enable effective navigation, understanding, and contribution.
---

# AGENTS.md

This file provides AI agents with comprehensive context about the operator-framework/api repository to enable effective navigation, understanding, and contribution.

## Project Overview

This repository contains the API definitions and validation libraries used by [Operator Lifecycle Manager][olm] (OLMv0). It's a foundational library in the [Operator Framework](https://github.com/operator-framework) ecosystem.

### Core Capabilities
- **API Definitions**: Kubernetes Custom Resource Definitions (CRDs) for OLM resources
- **Manifest Validation**: Static validators for operator bundles and package manifests
- **CLI Tool**: `operator-verify` for manifest verification
- **Common Libraries**: Shared utilities for bundle and manifest manipulation

## Custom Resource Definitions (CRDs)

| Resource | API Group | Description |
|----------|-----------|-------------|
| **ClusterServiceVersion (CSV)** | operators.coreos.com/v1alpha1 | Defines operator metadata, installation strategy, permissions, and owned/required CRDs |
| **Subscription** | operators.coreos.com/v1alpha1 | Tracks operator updates from a catalog channel |
| **InstallPlan** | operators.coreos.com/v1alpha1 | Calculated list of resources to install/upgrade |
| **CatalogSource** | operators.coreos.com/v1alpha1 | Repository of operators and metadata |
| **OperatorGroup** | operators.coreos.com/v1 | Groups namespaces for operator installation scope |
| **OperatorCondition** | operators.coreos.com/v2 | Tracks operator health status and conditions |

## Directory Structure

```
api/
├── cmd/                          # Entry point binaries
│   └── operator-verify/          # CLI tool for manifest verification
│
├── pkg/                          # Core implementation
│   ├── operators/                # OLM API types
│   │   ├── v1alpha1/             # Core OLM types (CSV, Subscription, etc.)
│   │   ├── v1/                   # OperatorGroup, OperatorCondition v1
│   │   ├── v2/                   # OperatorCondition v2
│   │   └── reference/            # Image reference parsing
│   │
│   ├── validation/               # Operator manifest validators
│   │   ├── errors/               # Validation error types
│   │   ├── interfaces/           # Validator interfaces
│   │   └── internal/             # Validator implementations
│   │
│   ├── manifests/                # Bundle and manifest loaders
│   │
│   ├── constraints/              # Constraint and CEL validation
│   │
│   ├── lib/version/              # Version utilities
│   │
│   └── apis/scorecard/           # Scorecard configuration types
│
├── crds/                         # Generated CRD YAML files
│
└── hack/                         # Build scripts and tools
```

## Key Packages and Their Responsibilities

### API Types (`pkg/operators/`)

Defines all Kubernetes custom resources used by OLM:
- `v1alpha1/`: Core types (CSV, Subscription, InstallPlan, CatalogSource)
- `v1/`: OperatorGroup, Operator, OperatorCondition
- `v2/`: OperatorCondition v2
- `reference/`: Container image reference parsing utilities

**Key files**:
- `v1alpha1/clusterserviceversion_types.go` - CSV API definition
- `v1/operatorgroup_types.go` - OperatorGroup API definition

### Validation (`pkg/validation/`)

Static validators for operator bundles and manifests:
- **Default Validators**: Required checks for all operators
- **Optional Validators**: Community, OperatorHub, and best practice validators
- **Custom Validators**: Extensible validator interface

**Key files**:
- `validation.go` - Main validator orchestration
- `internal/bundle.go` - Bundle structure validation
- `internal/csv.go` - CSV validation rules
- `internal/operatorhub.go` - OperatorHub requirements

**Validator Types**:
- `BundleValidator` - Bundle format and structure
- `CSVValidator` - ClusterServiceVersion validation
- `CRDValidator` - CRD validation
- `OperatorHubValidator` - OperatorHub.io requirements
- `GoodPracticesValidator` - Best practices checks
- `AlphaDeprecatedAPIsValidator` - Deprecated API detection

### Manifests (`pkg/manifests/`)

Bundle and package manifest loaders:
- Bundle loading from directories
- PackageManifest parsing
- Metadata extraction

**Key files**:
- `bundle.go` - Bundle representation and loading
- `bundleloader.go` - Bundle loading logic

## Development Workflow

### Building operator-verify CLI
```bash
make install            # Build and install operator-verify CLI
```

### Testing
```bash
make test-unit          # Run unit tests
make test               # Run all tests
make TEST=<name> test-unit  # Run specific test
```

### Code Generation
```bash
make generate           # Generate deep-copy methods
make manifests          # Generate CRD manifests
make verify             # Verify generated code is up-to-date
```

### Code Quality
```bash
make format             # Format source code
make tidy               # Update and verify dependencies
```

## Validation Usage

### Using Default Validators

```go
import (
    apimanifests "github.com/operator-framework/api/pkg/manifests"
    apivalidation "github.com/operator-framework/api/pkg/validation"
)

// Load bundle
bundle, err := apimanifests.GetBundleFromDir(path)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [operator-framework/api](https://github.com/operator-framework/api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
