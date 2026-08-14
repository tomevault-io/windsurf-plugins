---
trigger: always_on
description: This file gives repository-specific guidance to coding agents working in `strimzi-shutdown`.
---

# AGENTS.md

This file gives repository-specific guidance to coding agents working in `strimzi-shutdown`.
It is intentionally practical and biased toward the conventions already present in this codebase.

## Repository Snapshot

- Language: Go
- Module: `github.com/scholzj/strimzi-shutdown`
- Go version: `1.25.0`
- Dependency baseline: `github.com/scholzj/strimzi-go v0.8.0`, Kubernetes modules `v0.35.2`
- App type: small Cobra-based CLI for stopping and restarting Strimzi Kafka clusters
- Main packages:
  - `main.go` wires the CLI entrypoint
  - `cmd/` contains Cobra commands and shared Kubernetes helpers
- CI currently runs tests and a basic build only; there is no dedicated lint workflow

## Existing Agent Rules

- No repository-local Cursor rules were found in `.cursor/rules/`
- No `.cursorrules` file was found
- No Copilot instructions were found in `.github/copilot-instructions.md`
- Do not invent hidden policy files; rely on this document and the existing codebase

## Important Working Assumptions

- This is a CLI utility, not a long-running controller
- Behavior is user-facing through command output, so preserve clear log messages
- The project interacts with Kubernetes and Strimzi APIs directly through typed clients
- Backward compatibility matters for command names and existing flags
- The repository is small, so consistency with current files matters more than introducing frameworks

## Build Commands

- Build the default binary in the repo root:

```bash
go build
```

- Preferred local workflow via `Makefile`:

```bash
make build
```

- `make all` runs `build` first and `test` second

- `make build` runs formatting, vet, module tidy, and a verbose build:

```bash
gofmt -w main.go cmd/*.go
go vet -v ./...
go mod tidy -v
go build -v -o strimzi-shutdown
```

- Build all packages:

```bash
go build ./...
```

- Build a static-style binary like CI does for PR validation:

```bash
CGO_ENABLED=0 go build
```

- Build a named local binary:

```bash
go build -o strimzi-shutdown
```

- Cross-compile one target the same way the release workflow does:

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o strimzi-shutdown-linux-amd64
```

## Test Commands

- Run the full test suite:

```bash
go test ./...
```

- Preferred local test command via `Makefile`:

```bash
make test
```

- Run verbose tests, matching CI more closely:

```bash
go test -v ./...
```

- Run verbose tests with CGO disabled, matching PR CI exactly:

```bash
CGO_ENABLED=0 go test -v ./...
```

- Current state: there are no `*_test.go` files in the repository yet, so `go test ./...` reports `[no test files]`

## Running A Single Test

- Run one test by exact name in one package:

```bash
go test ./cmd -run '^TestName$'
```

- Run one subtest:

```bash
go test ./cmd -run '^TestParent$/^subcase$'
```

- Run one test across all packages when you are not sure where it lives:

```bash
go test ./... -run '^TestName$'
```

- Run one test with verbose output:

```bash
go test -v ./cmd -run '^TestName$'
```

- If you add tests for Kubernetes-facing helpers, prefer package-scoped runs first, then `go test ./...`

## Lint And Formatting Commands

- There is no checked-in lint config such as `.golangci.yml`
- A basic `Makefile` exists for common local workflows
- Minimum local hygiene before finishing changes:

```bash
gofmt -w main.go cmd/*.go
```

```bash
go vet ./...
```

- Tidy module metadata when dependencies or imports change:

```bash
go mod tidy -v
```

- `make build` already runs `gofmt`, `go vet -v`, `go mod tidy -v`, and `go build -v`
- `make clean` runs `go clean -x`, `go clean -testcache -x`, and removes local binaries

- If you touched imports heavily, use `gofmt` and rely on Go's default import grouping
- Do not introduce a new lint tool unless the user explicitly asks for it

## CI Behavior To Match

- `.github/workflows/pr.yaml` runs on Go `1.25.x` and executes:
  - `make build`
  - `make test`
- `.github/workflows/build.yaml` runs on Go `1.25.x` and executes:
  - `make build`
  - `make test`
  - multi-platform `go build` loops
  - Docker image build with `docker buildx`
- If you can only run a minimal verification locally, prioritize `go test ./...` and `go build ./...`

## Repository Structure Conventions

- Keep Cobra command definitions in `cmd/`
- Keep shared client and wait helper functions in `cmd/utils.go` unless a new helper file clearly improves clarity
- Keep `main.go` minimal; it should only bootstrap the root command
- Prefer extending existing command files over creating many tiny files unless the command becomes meaningfully complex

## Code Style Guidelines

## Formatting

- Follow standard Go formatting; use `gofmt`, not hand-aligned formatting
- Keep lines and blocks simple; this repository does not use dense helper abstractions
- Preserve the existing Apache license header style in Go files when creating new source files

## Imports

- Use Go's default import grouping enforced by `gofmt`
- Standard library imports come first, then a blank line, then external imports
- Use import aliases only when they add clarity or avoid collisions
- Existing examples to follow:
  - `kafkaapi` for Strimzi API package aliases
  - `metav1` for Kubernetes meta API imports

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scholzj/strimzi-shutdown](https://github.com/scholzj/strimzi-shutdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
