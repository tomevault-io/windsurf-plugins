---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tableau is a Go-based configuration converter that transforms Excel/CSV/XML/YAML files into protobuf-defined configuration files (JSON, Text, Bin). It uses Protocol Buffers (proto3) to define the structure of input data, extended with custom tableau options (field numbers 50000-99999 on `google.protobuf.*Options`).

## Common Commands

### Build & Run
```bash
go build ./...
go install github.com/tableauio/tableau/cmd/tableauc@latest
```

### Testing
```bash
# Run all unit tests
go test -v -timeout 30m -race ./...

# Run a single test
go test -v -run TestFunctionName ./path/to/package/

# Run functional tests (from repo root) — builds coverage-instrumented binary, runs it, compares golden files
./test/functest/run.sh

# Run benchmarks (profiling)
go test -bench=. ./test/bench/
go test -run ^Test_genConf$ -cpuprofile=cpu.prof ./test/bench/
go tool pprof -http :8888 cpu.prof
```

### Vet & Lint
```bash
go vet ./...

# Full lint (CI uses golangci-lint v2.2.1)
golangci-lint run

# Buf proto linting & build
buf lint
buf build
buf generate            # Regenerate Go code from proto files
buf dep update          # Update dependencies in buf.lock
```

### Error Code Generation
```bash
# Regenerate ecode_generated.go from i18n config (via go:generate directive in internal/tools/generate.go)
go generate ./internal/tools/
# Or regenerate all generated code
go generate ./...
```

## Architecture

### Two-Phase Generation Pipeline

The core pipeline has two phases, both orchestrated from `tableau.go`:

1. **protogen** (`internal/protogen/`): Reads Excel/CSV/XML/YAML workbooks and generates `.proto` files. Parses sheet headers (name row, type row, note row) to infer protobuf message structure with tableau-specific annotations.

2. **confgen** (`internal/confgen/`): Reads the same workbooks using the generated `.proto` definitions and produces output configuration files (JSON/Text/Bin). Validates data using `buf.build/go/protovalidate`.

Both generators use **concurrent processing** with hierarchical error collectors that cap errors at each level:
- protogen: generator(10) -> book(5) -> sheet(3)
- confgen: generator(20) -> book(10) -> sheet(5) -> message(3)

### Public API (`tableau.go`)

```go
// Full pipeline: proto + conf
Generate(protoPackage, indir, outdir string, setters ...options.Option) error

// Individual phases
GenProto(protoPackage, indir, outdir string, setters ...options.Option) error
GenConf(protoPackage, indir, outdir string, setters ...options.Option) error

// Generator constructors (for programmatic use)
NewProtoGenerator(protoPackage, indir, outdir string, options ...options.Option) *protogen.Generator
NewConfGenerator(protoPackage, indir, outdir string, options ...options.Option) *confgen.Generator

// Utilities
SetLang(lang string) error
NewImporter(workbookPath string) (importer.Importer, error)
GetVersionInfo() *VersionInfo
```

### Key Packages

| Package | Purpose |
|---------|---------|
| `cmd/tableauc/` | CLI tool (cobra). Modes: `default`, `proto`, `conf`. Supports YAML config file (`-c`). |
| `internal/protogen/` | Generates `.proto` from workbook structure. Table parser + document parser. |
| `internal/confgen/` | Generates config from workbooks + proto descriptors. Table parser + document parser. |
| `internal/confgen/fieldprop/` | Field property validation: unique, sequence, order, range, refer, presence. |
| `internal/importer/` | `Importer` interface with Excel/CSV/XML/YAML implementations. |
| `internal/importer/book/` | In-memory workbook/sheet/table/row/node representation. |
| `internal/importer/book/tableparser/` | Table header parsing (name/type/note rows, data rows). |
| `internal/importer/metasheet/` | `@TABLEAU` metasheet parsing and context. |
| `format/` | Input formats (Excel, CSV, XML, YAML) and output formats (JSON, Bin, Text). |
| `options/` | Functional options pattern. YAML-serializable. `NewDefault()` for defaults. |
| `load/` | Runtime: load generated config files back into protobuf messages. Supports patch/merge/replace modes. |
| `store/` | Runtime: serialize protobuf messages to JSON/Text/Bin files. |
| `proto/tableau/protobuf/` | Source `.proto` files. Published to BSR as `buf.build/tableauio/tableau`. |
| `proto/tableaupb/` | Generated Go code from protos (**do not edit manually**). |
| `log/` | Structured logging via zap. Pluggable driver interface (`log/driver/`). |
| `internal/x/xerrors/` | Hierarchical error collection, structured key-value errors, stack traces, error codes (E0001-E3003). |
| `internal/x/xfs/` | Filesystem utilities (subdir rewrite, path cleaning, permissions). |
| `internal/x/xproto/` | Protobuf helpers: value parsing, merge, patch, union detection, type info. |
| `internal/x/xproto/protoc/` | Protobuf compiler wrapper using `protocompile`. |
| `internal/strcase/` | CamelCase/snake_case conversion with configurable acronyms. |
| `internal/types/` | Type matching (map, list, well-known messages), regex patterns for type DSL. |
| `internal/localizer/` | i18n support (BCP 47 language tags: en, zh). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tableauio/tableau](https://github.com/tableauio/tableau) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
