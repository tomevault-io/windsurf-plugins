---
trigger: always_on
description: Guidance for coding agents working in this repository. Keep `AGENTS.md` and `CLAUDE.md` synchronized.
---

# Repository Agent Guidance

Guidance for coding agents working in this repository. Keep `AGENTS.md` and `CLAUDE.md` synchronized.

## Current Documentation

When a task asks about a library, framework, SDK, API, CLI tool, or cloud service, use the `ctx7` CLI to fetch current documentation, including for API syntax, configuration, migrations, setup, and library-specific debugging. This does not apply to refactoring, writing scripts from scratch, debugging business logic, code review, or general programming concepts.

1. Resolve the official library name with `npx ctx7@latest library <name> "<specific topic>"` and choose the best relevant `/org/project` ID.
2. Fetch docs with `npx ctx7@latest docs <libraryId> "<specific topic>"`. Use separate requests for distinct concepts, with at most three Context7 commands per question.
3. Use a versioned ID from the library output for version-specific questions. Do not put secrets in queries.
4. If Context7 fails with a quota error, report it and suggest `npx ctx7@latest login` or `CONTEXT7_API_KEY`. If it fails with a network or DNS error in the sandbox, retry outside the sandbox.

## Project Overview

Tableau is a Go-based configuration converter that transforms Excel/CSV/XML/YAML files into protobuf-defined configuration files (JSON, Text, Bin). It uses Protocol Buffers (proto3), extended with custom tableau options on `google.protobuf.*Options` (field numbers 50000-99999).

## Common Commands

### Build & Run
```bash
go build ./...
go install github.com/tableauio/tableau/cmd/tableauc@latest
```

### Testing
```bash
# Run all unit tests
go test -v -timeout 30m ./...

# Run a single test
go test -v -run TestFunctionName ./path/to/package/

# Run functional tests (from repo root) — builds coverage-instrumented binary, runs it, compares golden files
./test/functest/run.sh

# Run benchmarks (profiling)
go test -bench=. ./test/bench/
go test -run ^Test_genConf$ -cpuprofile=cpu.prof ./test/bench/
go tool pprof -http :8888 cpu.prof
```

On Windows, do not run Go tests with the `-race` flag. Go in this environment is built with CGO disabled, while the race detector requires CGO.

### Vet & Lint
```bash
go vet ./...

# Full lint (CI runs golangci-lint)
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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tableauio/tableau](https://github.com/tableauio/tableau) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
