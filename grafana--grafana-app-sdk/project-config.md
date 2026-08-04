---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The `grafana-app-sdk` is an SDK for developing apps for the Grafana App Platform. It provides:
- A CLI (`grafana-app-sdk`) for generating code and projects
- Libraries for building Kubernetes operator-based applications
- Integration with Kubernetes API machinery and client-go

The SDK follows Kubernetes patterns and conventions, treating custom resources as first-class objects with support for validation, mutation, conversion, and reconciliation.

## Build, Test, and Development Commands

### Standard Development Workflow

```bash
# Install all dependencies and run full verification
make                       # Equivalent to: deps, lint, test, build

# Run tests with coverage
make test                  # Run all tests across submodules
make coverage              # Generate HTML coverage report

# Build the CLI binary
make build                 # Output: target/grafana-app-sdk

# Install CLI to GOPATH/bin
make install

# Run linter
make lint                  # Uses golangci-lint v2.5.0

# Update go.mod and go.work files
make update-workspace
```

### Running Individual Tests

```bash
# Run a specific test
go test -v -run TestName ./package/path

# Run tests in a specific package
go test ./k8s/...
go test ./operator/...

# Run a single test with verbose output
go test -v -count=1 -run TestInformerCustomCache ./operator/
```

### Performance Benchmarking

The project has comprehensive benchmarks for performance-critical components (informers, caches, operators):

```bash
# Run all benchmarks
make bench

# Establish baseline before making optimizations
make bench-baseline

# After code changes, compare against baseline
make bench-compare         # Uses benchstat for statistical analysis

# Generate memory and CPU profiles
make bench-profile
go tool pprof target/profiles/mem.out   # Analyze memory profile
go tool pprof target/profiles/cpu.out   # Analyze CPU profile
```

**Optimization workflow**: Profile first to identify bottlenecks → Establish baseline → Make changes → Compare to validate improvement.

### Code Generation

```bash
# Regenerate code from CUE definitions (requires CLI)
make generate

# Regenerate test golden files (codegen tests)
make regenerate-codegen-test-files
```

## Architecture and Key Packages

### Core Package Structure

The SDK is organized into several key packages that mirror Kubernetes patterns:

- **`resource/`** - Core resource abstractions (Kind, Object, Schema, Client, Store)
  - Defines the fundamental types for working with custom resources
  - Provides typed and untyped object wrappers
  - Implements storage interfaces compatible with Kubernetes patterns

- **`k8s/`** - Kubernetes client implementations
  - `Client` - Main resource client for CRUD operations
  - `ClientRegistry` - Manages multiple clients for different resource types
  - Cache implementations in `k8s/cache/` (controller, reflector with watch-list support)
  - Webhook handlers (admission, validation, mutation, conversion)

- **`operator/`** - Operator/controller framework components
  - `Informer` implementations (CustomCache, Kubernetes native, concurrent)
  - Reconcilers and watchers (Simple, Opinionated, Concurrent)
  - Controller loops for reacting to resource changes
  - Caching strategies (in-memory, memcached)

- **`simple/`** - High-level simplified APIs
  - `App` - Main application builder with opinionated defaults
  - `Operator` - Simplified operator creation
  - Integrates health checks, metrics, and tracing out of the box

- **`app/`** - Application manifest and runner
  - App manifest parsing and validation (CUE-based)
  - Application lifecycle management

- **`codegen/`** - Code generation from CUE schemas
  - CLI implementation for `grafana-app-sdk generate`

### Design Patterns

The SDK supports three application architectures:

1. **Frontend-only**: Custom Kinds with basic validation, no backend
2. **Operator-based**: Backend with validation, mutation, conversion, and reconciliation hooks
3. **Custom API**: Full control with extension API server for custom endpoints

All patterns assume resources can be modified outside your UI (kubectl, gitops tools like Flux, backup tools like Velero).

### Kubernetes Integration

- Built on `k8s.io/client-go` and `k8s.io/apimachinery`
- Uses `k8s.io/apiserver` for extension API servers
- Implements controller-runtime-like informers and reconcilers
- Supports watch-list streaming via `WatchListClient` feature gate
- Compatible with CRDs (Custom Resource Definitions)

### Testing Patterns

- Tests follow table-driven test patterns (see `go.dev/wiki/TableDrivenTests`)
- Uses `testify` for assertions
- Mocks are typically implemented as interfaces in test files
- Performance benchmarks in `benchmark/` package use realistic object counts (10k-50k)

## Go Workspace Configuration

This repository uses Go workspaces (`go.work`) for multi-module development. The workspace includes:
- Main SDK module
- `logging/` submodule

When modifying dependencies, always run `make update-workspace` to sync workspace files.

## Version and Dependencies

- Go version: 1.24.0 (see go.mod)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/grafana-app-sdk](https://github.com/grafana/grafana-app-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
