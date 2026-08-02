---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`helm-schema` is a Go tool that generates JSON Schema (Draft 7) files for Helm chart values. It traverses directories to find `Chart.yaml` files, reads associated `values.yaml` files, parses special `@schema` annotations in comments, and generates `values.schema.json` files that enable IDE auto-completion and validation for Helm values.

## Build and Test Commands

### Build

```bash
# Build the binary
go build -o helm-schema ./cmd/helm-schema

# Build with goreleaser (for releases)
goreleaser release --snapshot --clean
```

### Test

```bash
# Run all tests
go test ./...

# Run tests for a specific package
go test ./pkg/schema
go test ./pkg/chart

# Run a specific test
go test ./pkg/schema -run TestTopoSort

# Run tests with verbose output
go test -v ./...

# Integration tests (requires helm-schema binary in tests/)
cd tests && ./run.sh
```

### Linting and Formatting

```bash
# Format code
go fmt ./...

# Tidy dependencies
go mod tidy
```

## Code Architecture

### High-Level Flow

1. **Chart Discovery** (`pkg/chart/searching/`): Recursively searches for `Chart.yaml` files starting from a root directory. Also extracts `.tgz` chart archives if found.

2. **Parallel Processing** (`cmd/helm-schema/main.go`): Uses a worker pool (2x CPU cores) to process charts concurrently. Each worker receives chart paths via a channel.

3. **Schema Generation** (`pkg/schema/worker.go`): For each chart, reads the `values.yaml` file and parses YAML with annotations to build a JSON Schema object.

4. **Annotation Parsing** (`pkg/schema/schema.go`): Parses `# @schema` and `# @schema.root` comment blocks to extract JSON Schema properties (type, description, enum, pattern, etc.).

5. **Dependency Resolution** (`cmd/helm-schema/main.go`): After all charts are processed, topologically sorts them by dependencies and merges dependency schemas into parent charts. Library charts (type: library) have their properties merged at the top level.

6. **Output** (`cmd/helm-schema/main.go`): Writes `values.schema.json` files to each chart directory.

7. **Annotation Mode** (`pkg/schema/annotate.go`): With `--annotate` / `-A` flag, writes inferred `@schema` type annotation blocks into `values.yaml` files for keys that don't already have them. This is a separate execution mode that modifies values files instead of generating JSON schema. Keys that already have `@schema` blocks are skipped.

### Key Components

#### Schema Parsing (`pkg/schema/schema.go`)

- **`ParseValues()`**: Main entry point that parses a values.yaml file and returns a Schema
- **`parseYamlNode()`**: Recursively traverses YAML nodes, extracting schema annotations and inferring types
- **Annotation blocks**: Comments between `# @schema` markers are parsed as YAML to extract JSON Schema properties
- **Root annotations**: Comments between `# @schema.root` markers apply to the root schema object itself
- **Type inference**: If no type is specified, the tool infers it from YAML tags (!!str, !!int, !!bool, etc.)

#### Worker Pattern (`pkg/schema/worker.go`)

- Workers pull chart paths from a channel and process them independently
- Each worker:
  1. Reads Chart.yaml
  2. For schema generation, finds all configured values files that exist for the chart and merges them in CLI order
  3. Parses merged values into a Schema
  4. Sends Result to results channel

- When multiple values files are present, later files override earlier files using Helm-style nested map merge precedence.
- `--annotate` and `--add-schema-reference` still operate on the first matching values file only; they do not merge multiple files.

#### Dependency Graph (`pkg/schema/toposort.go`)

- **TopoSort()**: Uses DFS-based topological sorting to ensure dependencies are processed before dependents
- Detects circular dependencies and can either fail or warn based on `allowCircular` flag
- Returns charts in dependency order (dependencies first, parents last)

#### Chart Models (`pkg/chart/chart.go`)

- **ChartFile**: Represents Chart.yaml structure
- **Dependency**: Represents a chart dependency with name, version, alias, condition

#### Schema Merging (in `main.go`)

- Regular dependencies: Nested under dependency name (or alias) in parent schema
- Library charts: Properties merged directly into parent schema at top level
- Import-values: Properties from dependency's `exports` section (or custom paths) merged into parent at specified location
- Conditional dependencies: If a dependency has a `condition` field, the corresponding boolean property is auto-created in the dependency's schema
- Skip validation flag (`-m`): Can disable strict validation for dependencies by setting `additionalProperties: true`

#### Import-Values Processing (in `main.go`)

The `processImportValues()` function handles Helm's `import-values` directive:

- **Simple form** (`import-values: [defaults]`): Imports from `exports.defaults` in dependency to parent's root
- **Complex form** (`import-values: [{child: "path", parent: "path"}]`): Explicit source and target paths

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dadav/helm-schema](https://github.com/dadav/helm-schema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
