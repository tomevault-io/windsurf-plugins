---
trigger: always_on
description: This file provides context for AI coding agents (Claude Code, Codex, Kiro, etc.) working on the config-file-validator project.
---

# AGENTS.md

This file provides context for AI coding agents (Claude Code, Codex, Kiro, etc.) working on the config-file-validator project.

## Project overview

Config File Validator is a Go CLI tool that recursively scans directories for configuration files, detects their format by extension or known filename, and validates syntax and schema. It supports 18 file formats and outputs results in multiple report formats.

- Module path: `github.com/Boeing/config-file-validator/v2`
- Go version: 1.26
- Binary: `cmd/validator/validator.go`
- License: Apache 2.0

## Architecture

```
cmd/validator/       CLI entrypoint, flag parsing, orchestration
pkg/validator/       Validator implementations (one file per format)
pkg/filetype/        FileType registry, extension/known-file mapping
pkg/finder/          Filesystem walker, gitignore support, filtering
pkg/reporter/        Output formatters (stdout, JSON, JUnit, SARIF, GitHub)
pkg/cli/             CLI engine: wires finder → validators → reporters
pkg/schemastore/     SchemaStore catalog lookup and caching
pkg/configfile/      .cfv.toml config file parser
pkg/tools/           Small utility functions
internal/generate/   Code generators (known files from Linguist)
```

## Prerequisites

- Go 1.26+ (see `go.mod`)
- golangci-lint v2 (`go install github.com/golangci/golangci-lint/v2/cmd/golangci-lint@latest`)
- Node.js ≥ 20 (only for documentation site)

## Data flow

```
CLI parses flags → builds Finder (with file types, excludes, gitignore)
  → Finder walks filesystem, returns []FileMetadata
  → cli.Run() validates each file (syntax → schema if available)
  → Reporter.Print() formats results to stdout or file
```

The `pkg/cli` package is the orchestrator. It accepts a Finder, validators come from the FileType registry, and reporters are passed in from the CLI entrypoint.

## Local pipeline

When asked to "run the pipeline", "run pre-checks", or "verify before push", run all of these in order. Every check must pass before submitting a PR.

```
go vet ./...
test -z "$(gofmt -s -l -e .)"
golangci-lint run ./...
go generate ./pkg/filetype/...
go build -o /dev/null cmd/validator/validator.go
go test -cover -coverprofile coverage.out ./...
go tool cover -func coverage.out | grep total
```

Coverage must be ≥ 90%.

For fast iteration on a single package, run its tests directly:

```
go test ./pkg/validator/...
go test ./pkg/reporter/...
go test ./pkg/finder/...
```

Save the full pipeline for the final check before pushing.

## Quick reference

```
go test -v -run TestFoo ./pkg/validator/...                          # Run one test
go test -count=1 ./pkg/validator/...                                 # Skip test cache
go build -o ./validator cmd/validator/validator.go && ./validator .   # Build and run locally
go test -bench=. -benchmem ./pkg/finder/...                          # Benchmark finder
```

## Adding a new validator

1. Create `pkg/validator/<format>.go` with a struct implementing `validator.Validator`:

```go
package validator

type FooValidator struct{}

var _ Validator = FooValidator{}

func (FooValidator) ValidateSyntax(b []byte) (bool, error) {
    // Parse b. Return (true, nil) on success or (false, err) on failure.
    // Wrap errors in &ValidationError{Err: ..., Line: ...} when position is available.
}
```

2. Optionally implement `SchemaValidator` and/or `JSONMarshaler` if the format supports schema validation.

3. Register the file type in `pkg/filetype/file_type.go`:
   - Add a package-level `var FooFileType = FileType{...}` with name, extensions, and validator instance.
   - Add the name → pointer entry to `fileTypeRegistry`.
   - Add the value to the `FileTypes` slice in `init()`.

4. Add test cases in `pkg/validator/validator_test.go`. Follow the existing table-driven style. Add fuzz targets if the parser handles untrusted input.

5. Add a test fixture directory if needed under the existing test infrastructure.

6. Run `go generate ./pkg/filetype/...` if the format has known filenames in GitHub Linguist.

7. Update documentation:
   - `website/docs/reference/supported-file-types.md`
   - `website/docs/guides/file-type-detection.md` (if the format has known filenames)
   - `CHANGELOG.md` under `[Unreleased]` → `Added`

## Adding a new reporter

1. Create `pkg/reporter/<name>_reporter.go` implementing the `Reporter` interface:

```go
package reporter

type FooReporter struct {
    outputDest string
    isQuiet    bool
}

func NewFooReporter(outputDest string, isQuiet bool) *FooReporter {
    return &FooReporter{outputDest: outputDest, isQuiet: isQuiet}
}

func (r *FooReporter) Print(reports []Report) error {
    // Format reports and write to stdout or outputDest file.
    // Use outputBytesToFile() for file output.
    // Respect r.isQuiet (suppress stdout when true and outputDest is set).
}
```

2. Wire it into the CLI in `cmd/validator/validator.go`:
   - Add the format name to the `getReporter` switch/map.
   - Update the usage text with the new format name.

3. Add tests in `pkg/reporter/reporter_test.go`.

4. Update documentation:
   - `website/docs/guides/output-reporters.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Boeing/config-file-validator](https://github.com/Boeing/config-file-validator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
