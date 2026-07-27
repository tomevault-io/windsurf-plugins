---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## IMPORTANT

Cargo-style formatting is required for all DSL errors.

## Build & Test Commands

```bash
# Build
go build ./cmd/alab

# Run all tests
go test ./...

# Run tests with race detector (requires gcc)
CGO_ENABLED=1 go test -race ./...

# Run integration tests (requires Docker)
task db-up                    # Start test databases
go test ./... -tags=integration -count=1
task db-down                  # Stop test databases

# Run specific test
go test -run TestName ./internal/package -v

# Lint
golangci-lint run ./...

# Format
go fmt ./...
```

**Using Task (Taskfile.yml):**

```bash
task build          # Build to ./bin
task test           # Run unit tests
task test-race      # Run with race detector
task test-all       # Run all tests including e2e (manages Docker)
task lint           # Run golangci-lint
task check          # Run lint + vuln + test
```

## Architecture Overview

AstrolaDB is a **documentation-oriented meta-model system** that captures higher-order system metadata through a declarative JavaScript DSL. It's fundamentally about modeling system intent across multiple dimensions — structural, behavioral, and operational — making this metadata consumable by generators and documentation tooling.

### What is Higher-Order System Metadata?

AstrolaDB schemas are **not just database schemas**. They encode:

- **Structural metadata** — tables, columns, types, relationships (traditional schema)
- **Lifecycle metadata** — entity states, workflow transitions, state machines
- **Access metadata** — role-based or attribute-based policy intents
- **Event metadata** — event contracts, payloads, publish targets
- **Service metadata** — bounded contexts, ownership, domain partitions
- **Deployment metadata** — regions, replicas, storage, topology hints

**Migrations are just one built-in core** that consumes structural metadata to generate SQL. Generators consume the full metadata graph to produce application code, documentation, infrastructure templates, and more.

### Three-Layer Architecture

```
┌─────────────────────────────────────────┐
│  User Layer (JavaScript DSL)            │  ← Users write schemas/generators
│  • table(), col.*, fn.*                 │
│  • gen(), render()                      │
└──────────────┬──────────────────────────┘
               │ parsed by
               ▼
┌─────────────────────────────────────────┐
│  Runtime Layer (Goja VM)                │  ← Sandboxed JS execution
│  • internal/runtime/sandbox.go          │
│  • internal/runtime/builder/*.go        │
└──────────────┬──────────────────────────┘
               │ produces
               ▼
┌─────────────────────────────────────────┐
│  Core Layer (Go)                        │  ← Built-in metadata consumers
│  • internal/engine - Migration core     │  (structural → SQL)
│  • internal/dialect - SQL generation    │
│  • internal/types - Type exports        │
│  • [Future: Generator registry]         │  (full metadata → code/docs/IaC)
└─────────────────────────────────────────┘
```

### Key Directories

- `cmd/alab/` - CLI entry point, command implementations
- `internal/runtime/` - **Goja JavaScript VM and DSL bindings**
  - `sandbox.go` - Secure JS execution environment
  - `builder/` - JavaScript DSL implementation (col.\*, table(), metadata builders)
  - `jserror.go` - JavaScript error parsing and formatting
- `internal/engine/` - **Migration core** (one of multiple built-in cores)
  - Consumes structural metadata to generate SQL migrations
- `internal/dialect/` - SQL generation for PostgreSQL, SQLite
- `internal/alerr/` - Structured error system with error codes
- `internal/cli/` - CLI output formatting (Rust/Cargo-style errors)
- `internal/ast/` - Schema AST representation (structural + higher-order metadata)
- `pkg/astroladb/` - Public Go API

## Critical Implementation Details

### JavaScript DSL Conventions

#### Naming Convention: snake_case

ALL DSL methods and properties use snake_case, NOT camelCase.

Examples:

- ✅ `col.primary_key()`, `col.read_only()`, `table.sort_by()`
- ❌ `col.primaryKey()`, `col.readOnly()`, `table.sortBy()`

This applies to:

- Column methods: `.belongs_to()`, `.created_at`, `.updated_at`
- Table methods: `.sort_by()`, `.searchable()`, `.filterable()`
- Higher-order metadata methods: `.lifecycle()`, `.policy()`, `.events()`, `.meta()`, `.deploy()`
- All future DSL additions

### JavaScript Runtime & Error Handling

**Unified Error Pipeline — All DSL Features:**

All JavaScript DSL features (table definitions with higher-order metadata, manual migrations, custom generators) execute through the **same Sandbox** and must follow the **same error pipeline**:

1. JS code executes in Goja VM → validation errors `panic(vm.ToValue(string))`
2. Goja captures the JS call site (line:col) in the Exception stack
3. `ParseJSError()` extracts line, column, and message from the Exception
4. `wrapJSError()` adjusts line offset, reads source from file, builds `*alerr.Error`
5. `cli.FormatError()` renders the Cargo-style output


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hlop3z/astroladb](https://github.com/hlop3z/astroladb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
