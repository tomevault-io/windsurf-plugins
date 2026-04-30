---
trigger: always_on
description: This file provides guidance to LLM tools when working with code in this repository.
---

This file provides guidance to LLM tools when working with code in this repository.

## Project Overview

**gendi** is a compile-time dependency injection container generator for Go. It reads YAML configuration files and generates type-safe, efficient container code with full compile-time validation—no runtime reflection.

Key characteristics:
- All dependencies are resolved and type-checked during code generation
- Generated code uses direct type assertions
- YAML-based declarative service definitions with imports and overrides
- Support for service lifecycle (shared/non-shared), decorators, tagged injection, and custom compiler passes

## Essential Commands

### Building and Testing
```bash
# Run all tests
go test ./...

# Build the CLI generator
go build ./cmd/gendi

# Run the generator manually
go run ./cmd/gendi --config=examples/basic/gendi.yaml --out=examples/basic/internal/di --pkg=di

# Regenerate all examples
just gen-examples
# OR
go generate ./...
```

### Running Examples
```bash
# Basic example
cd examples/basic && go generate && go run .

# Advanced example
cd examples/advanced && go generate && go run .

# Custom pass example
cd examples/custom-pass
go run ./tools/gendi --config=./cmd/gendi.yaml --out=./cmd --pkg=main
go run ./cmd
```

### Running a Single Test
```bash
# Run specific test
go test -run TestName ./path/to/package

# Example: run generator tests
go test -run TestGenerator ./generator

# Run with verbose output
go test -v -run TestName ./path/to/package
```

## Architecture

### Code Generation Pipeline

The generator follows a multi-stage pipeline:

1. **YAML Loading** (`yaml/` package)
   - Loads root YAML config with `yaml.LoadConfig()`
   - Resolves imports (supports glob patterns and relative paths)
   - Merges imported configs (later imports override earlier ones)
   - Resolves `$this` tokens to current package paths

2. **Compiler Passes** (`config.go`)
   - Optional transformation stage via `di.ApplyPasses()`
   - Passes implement `Pass` interface with `Process(*Config) (*Config, error)`
   - Used for custom conventions, auto-tagging, argument modification
   - See `examples/custom-pass` for practical implementation

3. **Type Resolution** (`typeres/` package)
   - Uses `golang.org/x/tools/go/packages` to load Go types
   - Resolves type strings to `go/types.Type`
   - Handles generic constructors with type arguments
   - Validates constructor signatures

4. **IR Building** (`ir/` package)
   - Multi-phase transformation from `di.Config` to IR `Container`
   - **Phase 1**: Build parameters, tags, and services
   - **Phase 2**: Resolve constructors, decorators, and dependencies
   - **Phase 3**: Validate (circular dependencies, type compatibility)
   - **Phase 4**: Optimize (prune unreachable services, optimize shared flags)
   - IR represents fully analyzed and validated dependency graph

5. **Code Generation** (`generator/` package)
   - Converts IR to Go source code
   - Renders: parameters map, container struct, getter methods
   - Manages imports with `ImportManager`
   - Handles identifier collision with `IdentGenerator`
   - Formats output with `go/format`

### Key Package Roles

- **`di` (root package)**: Core config types (`Config`, `Service`, `Parameter`, `Tag`) and `Pass` interface
- **`cmd/`**: CLI implementation with flag parsing and orchestration
- **`yaml/`**: YAML parsing, import resolution, `$this` token replacement
- **`ir/`**: Intermediate representation and multi-phase analysis
- **`generator/`**: Code generation from IR to Go source
- **`typeres/`**: Wrapper around `go/packages` for type resolution
- **`parameters/`**: Runtime parameter provider interface and implementations
- **`stdlib/`**: Pre-built factory functions for stdlib types (channels, HTTP clients, loggers)

### Important Files

- `config.go`: Core config structures and `Pass` interface
- `ir/builder.go`: IR construction orchestration
- `generator/generator.go`: Main generator entry point
- `cmd/cli.go`: CLI workflow (`Run()` and `Generate()`)

## Configuration Concepts

### Service References and Arguments

Constructor arguments use special syntax:
- `@service.id` - Reference to another service
- `@.inner` - Inner service (decorators only)
- `%param.name%` - Parameter reference
- `!tagged:tag.name` - Tagged services collection
- `!spread:@service` - Spread slice into variadic parameters
- `!spread:!tagged:tag` - Spread tagged collection into variadic parameters
- `!go:pkg.Var` - Go package-level variable or constant (e.g., `!go:os.Stdout`, `!go:log.LstdFlags`)
- `!field:@service.Field` - Field access on a service (e.g., `!field:@config.Host`, `!field:@config.Database.DSN`)
- `!field:!go:pkg.Symbol.Field` - Field access on a Go package-level variable (e.g., `!field:!go:http.DefaultClient.Timeout`)
- `@service.Method` - Method constructor
- `literal` - YAML scalar literal

### Service Lifecycle

- **Shared (singleton)**: Created once on first access, cached, thread-safe
- **Non-shared (factory)**: New instance on each access, no caching

### Decorators

Decorators wrap existing services using `@.inner` reference. Multiple decorators are ordered by `decoration_priority` (higher priority wraps first).

### Tagged Injection


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asp24/gendi](https://github.com/asp24/gendi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
