---
trigger: always_on
description: SPDX-FileCopyrightText: 2025 The Crossplane Authors <https://crossplane.io>
---

<!--
SPDX-FileCopyrightText: 2025 The Crossplane Authors <https://crossplane.io>

SPDX-License-Identifier: CC-BY-4.0
-->

# CLAUDE.md

This file provides guidance to AI Agents when working with code in this repository.

## Overview

Upjet is a code generation framework that transforms Terraform providers into Crossplane providers.  
It generates Kubernetes CRDs, reconciliation controllers, example manifests, and provider configuration from Terraform schemas.

The framework supports three Terraform execution modes:
- **Terraform CLI** (fork-based): Spawns terraform processes
- **Terraform Plugin SDK v2** (no-fork): Direct Go library calls
- **Terraform Plugin Framework** (no-fork): Protocol-based communication (protov6)

## Development Commands

### Building and Testing
```bash
# First-time setup: Initialize build submodule
make submodules

# Run linting and tests (run before opening PR)
make reviewable

# Build the project
make build

# Run unit tests
make test
# Or with specific flags:
go test -v ./pkg/...

# Run specific test
go test -v ./pkg/config -run TestExternalName

# Run linting
make lint

# Check Go modules are tidy
make modules.check

# Generate code
make generate
```

### Common Development Workflow
```bash
# 1. Make changes to Upjet code
# 2. Test in a provider using replace directive in provider's go.mod:
#    replace github.com/crossplane/upjet/v2 => ../upjet
# 3. Run make reviewable before committing
# 4. Open PR with example provider changes
```

## High-Level Architecture

The framework follows a layered pipeline:

```
Configuration Layer (pkg/config)
    ↓
Code Generation Pipeline (pkg/pipeline)
    ↓
Type Generation (pkg/types, pkg/schema)
    ↓
Runtime System (pkg/controller, pkg/terraform, pkg/resource)
```

### Key Architectural Layers

**1. Configuration Layer (pkg/config)**
- `Provider`: Root configuration mapping Terraform schemas to CRDs
- `Resource`: Per-resource configuration (external names, references, sensitivity)
- `ExternalName`: Maps Terraform IDs ↔ Crossplane external names (required for all resources)

**2. Code Generation Pipeline (pkg/pipeline)**
- `Run()`: Main orchestrator that generates all code
- `CRDGenerator`: Creates `*_types.go` files with Kubernetes CRD structs
- `ControllerGenerator`: Creates `*_controller.go` files with reconciliation logic
- `TerraformedGenerator`: Creates `*_terraformed.go` files implementing resource.Terraformed interface
- `ExampleGenerator`: Creates YAML example manifests

**3. Schema/Type Transformation (pkg/schema, pkg/types)**
- `TypeBuilder`: Converts Terraform schemas recursively to Go types
- Field classification:
  - **ForProvider**: User-configurable parameters
  - **InitProvider**: Late-initialized fields
  - **AtProvider**: Read-only observations

**4. Runtime Layer (pkg/controller, pkg/terraform)**
- `Connector`: Creates workspace and ExternalClient per resource
- `ExternalClient`: Implements Observe/Create/Update/Delete operations
- `Workspace`: Per-resource Terraform working directory managing state files
- Three execution modes switch on resource configuration:
  - CLI: Spawns terraform binary (pkg/controller/external.go)
  - SDK v2: Direct library calls (pkg/controller/external_tfpluginsdk.go)
  - Framework: Protocol calls (pkg/controller/external_tfpluginfw.go)

## Critical Patterns

### External Names (Required)
Every resource MUST have an external name configuration. This is how Crossplane identifies resources in the remote system:

```go
// Common patterns:
config.NameAsIdentifier           // Uses "name" field as identifier
config.IDAsExternalName           // Uses Terraform "id" as identifier
config.TemplatedStringAsIdentifier("field", "{{ .parameters.x }}")
```

The external name is:
- Removed from spec (not user-provided)
- Extracted from Terraform state after creation
- Used for terraform import operations

### Cross-Resource References
Enable Kubernetes-style references between resources:

```go
r.References["subnet_ids"] = config.Reference{
    TerraformName: "aws_subnet",
    Extractor: "status.atProvider.id",  // Optional, defaults to external name
}
// Generates: SubnetIDRefs and SubnetIDSelector fields in CRD
```

### Late Initialization
Server-generated fields are automatically populated into spec after creation. Configure ignored fields:

```go
r.LateInitializer.IgnoredFields = []string{"tags", "metadata"}
```

### Async Operations
For resources with long create/delete times:

```go
r.UseAsync = true  // Enables concurrent async operations
```

### Management Policies
Control which operations are allowed (Crossplane v1.11+):
- Observe: Read-only
- Create/Update/Delete: Allow modifications
- LateInitialize: Auto-fill from server
- * (All): Default

## Code Generation Flow

```go
// 1. Extract Terraform schema
// terraform providers schema --json > schema.json

// 2. Create provider configuration
provider := config.NewProvider(
    schemaJSON,
    "aws",                              // Terraform resource prefix
    "github.com/upbound/provider-aws",
    metadataYAML,
    config.WithIncludeList([]string{"aws_vpc", "aws_subnet"}),
)

// 3. Configure resources
provider.AddResourceConfigurator("aws_vpc", func(r *config.Resource) {
    r.ExternalName = config.NameAsIdentifier

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crossplane/upjet](https://github.com/crossplane/upjet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
