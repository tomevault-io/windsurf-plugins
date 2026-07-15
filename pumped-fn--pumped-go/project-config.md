---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Pumped Go is a graph-based dependency injection and reactive execution library for Go. It organizes code around three core concepts:

1. **Executors**: Units of computation with explicit dependencies (long-lived resources)
2. **Scopes**: Lifecycle managers that resolve and cache executor values
3. **Flows**: Short-span executable operations with hierarchical execution contexts

## Important Documentation

Before working on this codebase, read these key documents:

- **DESIGN.md** - Core design decisions, TypeScript vs Go differences, why controllers for everything, performance considerations
- **INTEGRATION_PATTERNS.md** - Detailed comparison of CLI vs HTTP server integration patterns, when caching matters, hybrid patterns for workers

## Code Quality Requirements

**CRITICAL: When making any code changes, you MUST ensure the following pass before committing:**

1. **Linting**: `devbox run lint` or `golangci-lint run --timeout=5m`
2. **Build**: `devbox run build` or `go build -v ./...`
3. **Tests**: `devbox run test` or `CGO_ENABLED=1 go test -v -race ./...`

**Workflow for code changes:**
```bash
# 1. Make your changes
# 2. Run all three checks
devbox run lint
devbox run build
devbox run test

# OR run full CI pipeline
devbox run ci

# 3. Only commit if ALL checks pass
```

**Do NOT commit code that:**
- Fails linting checks
- Fails to build
- Breaks existing tests
- Introduces race conditions (detected by `-race` flag)

## Development Commands

This project uses [Devbox](https://www.jetify.com/devbox/) for reproducible development environments.

### Setup
```bash
devbox shell                 # Enter dev environment
devbox run setup            # Initial setup
```

### Testing
```bash
devbox run test             # Run tests with race detection
devbox run test-coverage    # Run with coverage report
devbox run coverage         # View coverage percentages
devbox run benchmark        # Run benchmarks
devbox run integration-test # Run integration tests (-tags=integration)
```

### Code Quality
```bash
devbox run lint             # Run golangci-lint
devbox run lint-fix         # Auto-fix linting issues
devbox run fmt              # Format code
devbox run vet              # Run go vet
devbox run security         # Run gosec security scanner
```

### Building
```bash
devbox run build            # Build library
devbox run build-examples   # Build all example applications
```

### CI/CD
```bash
devbox run ci               # Full CI pipeline (deps, lint, test, build)
devbox run pre-commit       # Pre-commit checks (fmt, lint, test)
devbox run release-snapshot # Test release locally
devbox run release-test     # Validate release config
```

### Running a Single Test
```bash
# Run specific test
CGO_ENABLED=1 go test -v -race -run TestName ./...

# Run tests in specific file
CGO_ENABLED=1 go test -v -race ./path/to/package

# Run single test in package
CGO_ENABLED=1 go test -v -race -run TestSpecificName ./executor_test.go
```

## Architecture

### Core Design Principles

1. **Executors are their own keys**: No string IDs required - executor pointers serve as unique identifiers
2. **Controllers for everything**: All dependencies passed as `*Controller[T]` for maximum flexibility (get, update, reload, release)
3. **Standalone functions for type parameters**: Since Go doesn't support type params on methods, use `Resolve[T](scope, exec)` instead of `scope.Resolve[T](exec)`
4. **Reactive graph tracking**: Dependencies tracked via `ReactiveGraph` (graph.go) for efficient invalidation propagation

### Key Files

- **executor.go**: Defines `Executor[T]`, `Dependency`, `DependencyMode` (Static/Reactive/Lazy)
- **scope.go**: Scope lifecycle, resolution, caching, reactive graph management
- **flow.go**: Flow execution, `ExecutionCtx`, execution tree tracking
- **graph.go**: `ReactiveGraph` for tracking upstream/downstream dependencies
- **controller.go**: `Controller[T]` providing Get/Update/Reload/Release operations
- **extension.go**: Extension interface for cross-cutting concerns
- **tag.go**: Type-safe metadata system
- **executor_generated.go, flow_generated.go**: Generated code for Derive1-5, Flow0-5 (see codegen/)

### Dependency Modes

- **Static** (default): Resolve once, cache forever
- **Reactive**: Invalidate and re-resolve when dependency changes
- **Lazy**: Defer resolution until explicitly requested

See DESIGN.md for detailed explanations and performance considerations.

### Reactive Graph Traversal

The reactive graph (graph.go) uses **iterative traversal** (not recursive) to find all dependents when an executor is updated. This prevents stack overflow on deep dependency chains.

```go
// In scope.go:
func (s *Scope) findReactiveDependents(exec AnyExecutor) []AnyExecutor {
    return s.graph.FindDependents(exec) // Iterative traversal in graph.go
}
```

### Controllers

All dependencies are passed as controllers. Key methods: `Get()`, `Peek()`, `Update()`, `Release()`, `Reload()`, `IsCached()`. See DESIGN.md for why controllers are used for everything.

### Flows vs Executors


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pumped-fn/pumped-go](https://github.com/pumped-fn/pumped-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
