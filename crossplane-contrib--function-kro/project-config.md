---
trigger: always_on
description: This document provides comprehensive orientation for AI agents and developers working with the function-kro codebase.
---

# Function-KRO Agent Guide

This document provides comprehensive orientation for AI agents and developers working with the function-kro codebase.

## What is Function-KRO?

Function-KRO is a **Crossplane Composition Function** that brings the KRO (Kubernetes Resource Orchestration) experience to Crossplane users. It enables declarative definition and management of complex, interdependent Kubernetes resources using CEL (Common Expression Language) expressions.

### Purpose

The main goal is to enable Crossplane users to have the **KRO experience alongside their other Crossplane functions** and choose it when it makes the most sense for their needs. This provides:

- **Declarative Resource Dependencies**: Define resources that depend on outputs from other resources using CEL expressions
- **Conditional Resource Creation**: Include or exclude resources based on CEL conditions (`includeWhen`)
- **Readiness Conditions**: Define when a resource is considered ready (`readyWhen`)
- **Status Aggregation**: Aggregate data from composed resources back to the XR status
- **Type-Safe Templates**: Full CEL type checking against OpenAPI schemas

### Relationship to KRO

KRO is a standalone Kubernetes controller for resource orchestration. Function-KRO adapts KRO's core logic to run as a Crossplane composition function:

| Standalone KRO | Function-KRO |
|----------------|--------------|
| Watches ResourceGraph CRs directly | Receives input via Crossplane function requests |
| Gets schemas from Kubernetes API server | Gets schemas from Crossplane's schema resolution |
| Creates resources directly | Returns desired resources to Crossplane |

## Architecture Overview

### Request Processing Flow

```
Crossplane RunFunctionRequest
    ↓
┌──────────────────────────────────────────────────────────────────┐
│ fn.go: RunFunction()                                             │
│                                                                  │
│  1. Parse ResourceGraph input                                    │
│  2. Request schemas (requireSchemas → buildResolver)             │
│  3. Build resource graph (graph.NewBuilder → NewResourceGraphDef)│
│  4. Create runtime (runtime.FromGraph)                           │
│  5. Process external refs:                                       │
│     - Resolve identity (GetDesiredIdentity with CEL)             │
│     - Request from Crossplane, inject as observed state          │
│  6. Set observed composed resources on runtime nodes             │
│  7. Process nodes in topological order:                          │
│     - Check IsIgnored (includeWhen + contagious dependency skip) │
│     - Evaluate GetDesired (hard resolve for resources/collections│
│       soft resolve for instance)                                 │
│     - Handle collections (forEach → {id}-{name}, ...)            │
│     - Evaluate readiness (CheckReadiness → error sentinel)      │
│  8. Build XR status from instance node's soft-resolved fields    │
└──────────────────────────────────────────────────────────────────┘
    ↓
Crossplane RunFunctionResponse (desired resources + XR status)
```

### Key Components

```
fn.go                        Main function entry point
    ↓
kro/graph/builder.go         Builds validated resource graphs
    ├── node.go              Node types (Resource, Collection, External, Instance)
    ├── parser/              Extracts CEL expressions from templates
    ├── fieldpath/           Field path building and parsing
    ├── schema/resolver/     OpenAPI schema resolution strategies
    ├── dag/                 Directed acyclic graph for dependencies
    ├── variable/            CEL variable management
    └── validation.go        Validates resources and expressions
    ↓
kro/runtime/                 Executes the graph, evaluates CEL
    ├── runtime.go           Runtime interface, FromGraph constructor
    ├── node.go              Runtime node (GetDesired, CheckReadiness, IsIgnored, etc.)
    ├── node_resolve.go      Resolution helpers (soft/hard resolve, collections)
    ├── collection.go        Collection identity matching, index labels
    └── resolver/            Template field resolution (path-based value injection)
    ↓
kro/cel/                     CEL environment setup and evaluation
    ├── environment.go       CEL environment construction with options
    ├── ast/inspector.go     AST inspection for dependency extraction
    ├── library/random.go    Custom CEL random functions
    └── schemas.go           Schema-to-CEL type mapping
```

## Codebase Structure

```
function-kro/
├── fn.go                    # Main function implementation (START HERE)
├── main.go                  # CLI entry point with gRPC server setup
├── spec-desired-ssa.md      # SSA design spec (historical — see note in Design Decisions)
│
├── input/v1alpha1/          # Input API definitions (ResourceGraph type)
│
├── kro/                     # Core KRO implementation
│   ├── cel/                 # CEL environment setup and expression evaluation
│   │   ├── ast/             # AST inspection for dependency extraction
│   │   └── library/         # Custom CEL functions (random)
│   ├── features/            # Feature gate definitions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crossplane-contrib/function-kro](https://github.com/crossplane-contrib/function-kro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
