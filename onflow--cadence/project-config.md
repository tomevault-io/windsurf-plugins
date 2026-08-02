---
trigger: always_on
description: Guidance for AI coding agents working in this repository. Every rule below traces to a file in the repo.
---

# AGENTS.md

Guidance for AI coding agents working in this repository. Every rule below traces to a file in the repo.

## Overview

Reference Go implementation of Cadence, Flow's resource-oriented smart contract language (`module github.com/onflow/cadence`). The repo ships the lexer, parser, semantic checker, tree-walking interpreter, a bytecode compiler+VM (`bbq/`), runtime, stdlib, CLI tools, and the `@onflow/cadence-parser` npm package. License: Apache 2.0.

## Build and Test Commands

Use `go 1.24.0` locally (per `go.mod`); `.github/workflows/ci.yml` pins `go-version: '1.25'` for the `commands-and-tools` job.

- `make build` — builds CLI binaries (`cmd/parse/parse`, `cmd/parse/parse.wasm`, `cmd/check/check`, `cmd/main/main`) plus `build-tools` (analysis, get-contracts, compatibility-check).
- `make test` — runs `test-with-compiler` + `test-with-tracing`, then `go test -tags compare_subtyping $(TEST_PKGS)`. `TEST_PKGS` excludes `/cmd`, `/analysis`, `/tools` (Makefile line 91).
- `make test-with-compiler` — `go test ./interpreter/... ./runtime/... -compile=true` (routes the suite through the `bbq` VM).
- `make test-with-tracing` — same packages with `-tags cadence_tracing`.
- `make test-with-coverage` — `-race -coverprofile=coverage.txt -covermode=atomic`; post-processes `coverage.txt` via `sed` to drop zero-line entries.
- `make smoke-test` — `go test -count=5 ./interpreter/... -runSmokeTests=true -validateAtree=false`.
- `make ci` / `make ci-with-tracing` — workflow targets invoked by `.github/workflows/ci.yml`.
- `make test-tools` — `test-analysis test-compatibility-check test-subtype-gen`.
- `make bench` / `make bench-common` — `go test -bench=. -benchmem -shuffle=on -benchtime=$(BENCH_TIME)`. `BENCH_REPS` and `BENCH_TIME` are overridable; install benchstat via `make install-benchstat`.
- `make lint` — builds `tools/golangci-lint/golangci-lint` plus the custom linter `.so` plugins, then runs with `--timeout=5m -v ./...`. `make fix-lint` adds `--fix`.
- `make generate` — installs `stringer@v0.32.0`, then `go generate -v ./...`. Required after touching any file with a `//go:generate` directive.
- `make check-headers` — runs `./check-headers.sh` (fails on `.go` files missing the Apache 2.0 header or a `Code generated` marker).
- `make check-tidy` — runs `generate`, then `go mod tidy`, then `git diff --exit-code`.
- `make validate-error-doc-links` — `go run ./cmd/errors validate-doc-links`.
- `make release bump=patch|minor|major` — drives `bump-version.sh`, updating `version.go` and `npm-packages/cadence-parser/package.json`.

Concurrent-checker smoke test (from `docs/development.md`): `go test -race -v ./sema/... -cadence.checkConcurrently=10` — runs each sema test 10 times concurrently and asserts error equality.

## Architecture

Compiler/runtime is a pipeline of Go packages at the repo root:

- `ast/` — AST node types; many files pair with `*_string.go` generated via `stringer`.
- `parser/`, `old_parser/` — current parser and the pre-1.0 parser retained for contract-upgrade compatibility checks.
- `sema/` — semantic analyzer / type checker. Contains generators under `sema/gen/` and `sema/type_check_gen/`.
- `interpreter/` — tree-walking interpreter; includes generated tables (`subtype_check.gen.go`, `type_check_gen/`).
- `bbq/` — bytecode pipeline: `bbq/compiler`, `bbq/vm`, `bbq/opcode`, `bbq/constant`, `bbq/leb128`, `bbq/commons`, `bbq/test_utils`. Activated in tests by the `-compile=true` flag.
- `runtime/` — embedding API: `environment.go`, CCF/JSON value conversion, storage. Contract-update validation tests live here (`contract_update_validation_test.go`); the implementation is `stdlib/contract_update_validation.go`.
- `stdlib/` — built-in contracts/functions (crypto, RLP under `stdlib/rlp/`, test framework, PublicKey, `contract_update_validation.go`); `stdlib/contracts/` holds embedded Cadence source.
- `common/`, `errors/`, `encoding/ccf`, `encoding/json`, `values/`, `format/`, `pretty/`, `fixedpoint/`, `integer/`, `activations/` — supporting packages.
- `cmd/` — CLI entrypoints: `parse`, `check`, `main` (REPL/executor), `execute`, `info`, `lex`, `json-cdc`, `minifier`, `decode-slab`, `decode-state-values`, `errors`. Usage examples in `docs/development.md`.
- `tools/` — Go tooling plus custom linters (`maprange`, `unkeyed`, `constructorcheck`), a `golangci-lint` wrapper, `analysis`, `ast-explorer`, `compare-parsing`, `compatibility-check`, `get-contracts`, `go-apply-expr-diff`, `pretty`, `storage-explorer`, `subtype-gen`, `staged-contracts-report-printer`, `update`, `accounts-script`.
- `compat/` — Python contract-compatibility suite (`pyproject.toml`, `main.py`, `requirements.txt`); driven by `.github/workflows/compat.yaml`.
- `fuzz/` and `fuzz.go` — go-fuzz entry point that calls `parser.ParseProgram` then constructs a `sema` checker.
- `npm-packages/cadence-parser` — `@onflow/cadence-parser` v1.10.2; `npm run build` compiles `cmd/parse` to WASM (`GOARCH=wasm GOOS=js`).
- `test_utils/` — shared fixtures (`common_utils`, `sema_utils`, `runtime_utils`, `interpreter_utils`, `contracts`). Dot-imports are whitelisted only for these paths.

## Conventions and Gotchas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onflow/cadence](https://github.com/onflow/cadence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
