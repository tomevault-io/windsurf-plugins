---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Promener is a code generator for Prometheus metrics that creates type-safe, organized code from CUE specifications. It generates structured metrics code organized by namespace/subsystem, with CEL-based runtime label validation, optional Uber FX dependency injection support, and HTML documentation generation. CUE is used as the source of truth, providing both data definition and validation in one declarative language.

## Build and Development Commands

### Building
```bash
go build                           # Build the binary (outputs to ./promener)
go build -o promener              # Build with explicit output name
```

### Testing
```bash
go test ./...                      # Run all tests
go test ./internal/parser          # Run parser tests only
go test ./internal/generator       # Run generator tests only
go test ./internal/domain          # Run domain tests only
go test -v ./...                   # Run tests with verbose output
```

### Running
```bash
# Validate CUE specification (schema is embedded)
./promener vet testdata/test.cue

# Validate with JSON output for CI/CD
./promener vet testdata/test.cue --format json

# Generate Go metrics code
./promener generate go -i testdata/test.cue -o ./out

# Generate with FX dependency injection
./promener generate go -i testdata/test.cue -o ./out --di --fx

# Generate .NET code
./promener generate dotnet -i testdata/test.cue -o ./out

# Generate Node.js/TypeScript code
./promener generate nodejs -i testdata/test.cue -o ./out

# Generate HTML documentation
./promener html -i testdata/test.cue -o docs/metrics.html

# Override package name
./promener generate go -i metrics.cue -o ./out -p mymetrics
```

### Development with go generate
```bash
go generate ./...                  # Run all go:generate directives
```

## Example CUE Specification

A complete example with label validations:

```cue
package main

version: "1.0.0"
info: {
    title:   "API Metrics"
    version: "1.0.0"
}
services: {
    default: {
        info: {
            title:   "Default Service"
            version: "1.0.0"
        }
        metrics: {
            http_requests_total: {
                namespace: "http"
                subsystem: "server"
                type:      "counter"
                help:      "Total HTTP requests"
                labels: {
                    method: {
                        description: "HTTP method"
                        validations: [
                            "value in ['GET', 'POST', 'PUT', 'DELETE', 'PATCH']",
                        ]
                    }
                    status: {
                        description: "HTTP status code"
                        validations: [
                            "value.matches('^[1-5][0-9]{2}$')",
                        ]
                    }
                    service: {
                        description: "Service name (DNS-compatible)"
                        validations: [
                            "value.matches('^[a-z][a-z0-9-]*$')",
                            "size(value) >= 3",
                            "size(value) <= 63",
                        ]
                    }
                }
            }
        }
    }
}
```

This generates type-safe Go code with runtime validation:
```go
metrics := metrics.Default()
metrics.Http.Server.IncRequestsTotal("GET", "200", "api-gateway")
```

## Architecture

### Core Components

**Domain Models** (`internal/domain/`)
- `Specification`: Top-level structure representing the complete spec (OpenAPI-inspired format)
- `Metric`: Individual metric definition with namespace, subsystem, type, labels, and optional constant labels
- `Labels`: Flexible label definitions supporting both simple string arrays and detailed maps with descriptions and CEL validations
- `LabelDefinition`: Individual label with name, description, and optional CEL validation expressions
- `Validation`: CEL-based validation with compiled programs for runtime enforcement
- `ConstLabels`: Static labels with support for environment variable substitution (e.g., `${ENVIRONMENT:production}`)
- `MetricType`: Counter, Gauge, Histogram, Summary
- All domain types include validation methods

**Generator** (`internal/generator/`)
- Transforms `domain.Specification` into Go code using text templates
- Organizes metrics hierarchically: `metrics.{Namespace}.{Subsystem}.Method()`
- Two main templates:
  - `registry_template.go`: Main metrics registry with type-safe methods and CEL validation
  - `fx_template.go`: Uber FX dependency injection module (optional via `--fx` flag)
- `model.go`: Builds template data by grouping metrics into namespaces/subsystems with label definitions
- `envvar.go`: Handles environment variable substitution in constant labels
- Generates thread-safe initialization using `sync.Once`
- Compiles CEL validation expressions at initialization (one-time cost)
- Validates label values before each metric operation (panics on validation failure)
- Uses `go/format` to format generated code

**HTML Generator** (`internal/htmlgen/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jycamier/promener](https://github.com/jycamier/promener) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
