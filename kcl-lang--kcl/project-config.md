---
trigger: always_on
description: This document provides a comprehensive overview of the KCL repository to help AI assistants understand the codebase for future work.
---

# KCL Repository Overview for AI Assistants

This document provides a comprehensive overview of the KCL repository to help AI assistants understand the codebase for future work.

## What is KCL?

**KCL (KCL Constraint-based Record & Functional Language)** is an open-source configuration and policy language designed for cloud-native scenarios. It's a CNCF Sandbox project that enhances the writing of complex configurations through advanced programming language technology.

### Main Purpose
- Generate low-level static configuration data (JSON, YAML) for cloud-native applications
- Reduce boilerplate in configuration through schema modeling
- Define and validate configuration data with rule constraints
- Manage large-scale configurations with GitOps and automation
- Mutate/validate Kubernetes resources through various tool plugins
- Platform engineering language for modern application delivery (used with KusionStack)

### Key Use Cases
- Kubernetes configuration management and abstraction
- Terraform resource model abstraction
- Configuration validation and constraint checking
- Large-scale infrastructure as code
- Platform engineering and GitOps workflows

### Production Users
Ant Group, Youzan, and Huawei are notable production users managing large-scale Kubernetes deployments.

## Repository Structure

### Top-Level Organization

```
/crates/              Core KCL VM and compiler implementation (main codebase)
/compiler_base/      Base compiler libraries and utilities (WIP, rustc-derived)
/cli/                Command-line interface binary wrapper
/test/               Integration and grammar tests
  /grammar/          Extensive grammar test cases
  /integration/      Integration test suites
/docs/               Developer guides and documentation
  /dev_guide/        Development guide (architecture, quick start, etc.)
  /design/           Design documents
/scripts/            Build and release automation scripts
/.github/workflows/  CI/CD pipelines for multiple platforms
```

### Key Files
- `Makefile` - Top-level build orchestration
- `LICENSE` - Apache License 2.0
- `README.md` - Project documentation

## Technology Stack

### Primary Language: Rust
- 362+ Rust source files
- ~32,673 lines of Rust code in core modules
- **Requires Rust 1.91+** for building
- Rust 2024 edition

### Secondary Languages
- **KCL** - The language itself (.k files for examples and tests)
- **Python** - Integration tests and test infrastructure
- **Shell/Bash** - Build scripts and automation
- **C/C++** - Runtime interop and FFI interfaces

### Key Dependencies
- **Protobuf** - API definitions and RPC communication
- **WASM** - WebAssembly compilation target support
- **tokio** - Async runtime (for LSP and server)
- **salsa** - Incremental computation (for LSP)

## Architecture

### Compilation Pipeline

```
Source Code (.k files)
    ↓
[Lexer] → Tokens
    ↓
[Parser] → AST
    ↓
[Resolver/Sema] → Semantic Analysis & Type Checking
    ↓
[Evaluator/Runner] → Execution
    ↓
Output (YAML/JSON)
```

### Key Components (/crates)

**Frontend (Parsing & Analysis):**
- `kcl-lexer` - Lexical analysis and tokenization
- `kcl-parser` - Parse KCL source into AST
- `kcl-ast` - Abstract Syntax Tree definitions and walker
- `kcl-ast-pretty` - AST formatting and pretty-printing
- `kcl-span` - Source code span/position tracking
- `kcl-error` - Error handling and diagnostics

**Semantic Analysis:**
- `kcl-sema` - Semantic analysis, type checking, and validation
- `kcl-loader` - Module loading and dependency management
- `kcl-query` - Code query and information retrieval

**Compilation & Execution:**
- `kcl-evaluator` - Expression evaluation engine
- `kcl-runner` - Program execution environment
- `kcl-driver` - Compilation driver and orchestration

**Runtime:**
- `kcl-runtime` - Runtime support libraries with extensive standard library
  - Value representation and type system
  - Standard library modules: json, yaml, base64, regex, crypto, datetime, math, net, etc.
  - Template rendering (handlebars)
  - File I/O and manifests

**Tooling:**
- `kcl-tools` - Development tools
  - Format, Lint, Fix, Vet
  - Testing infrastructure
  - **LSP** (Language Server) - Full IDE support with autocomplete, goto-definition, diagnostics
- `kcl-api` - Public API layer for multi-language SDKs
- `kcl-cmd` - CLI command implementation

**Utilities:**
- `kcl-config` - Configuration parsing
- `kcl-version` - Version management
- `kcl-utils` - Common utilities
- `kcl-primitives` - Primitive type definitions
- `kcl-macros` - Procedural macros

### Language Server Architecture
- Salsa-based incremental compilation for performance
- VFS (Virtual File System) for handling unsaved changes
- Thread pool for concurrent request handling
- Event-driven architecture (Tasks + LSP Messages)
- Compile unit discovery for projects without explicit config
- Located at: `/crates/tools/src/LSP`

## Build System

### Build Tools
- **Cargo** - Primary Rust build system (workspace-based with 20+ crates)
- **Make** - Top-level orchestration
- **Docker** - Containerized build environment (recommended: `kcllang/kcl-builder`)

### Common Build Commands

```bash
make build          # Standard build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kcl-lang/kcl](https://github.com/kcl-lang/kcl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
