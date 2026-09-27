---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Checker is a zero-dependency Go library (`github.com/cinar/checker/v2`) for validating and normalizing user
input, driven by struct tags (`checkers:"..."`) or plain function calls. It ships 23 translated locales, JSON
Schema generation from checker tags, separately-versioned `gin`/`echo`/`nethttp`/`fiber` framework adapter
modules, an opt-in `nfkc` checker module, a `checkerlint` static analyzer, a `checkergen` reflection-free
code generator, and a standalone `cli` command-line interface.

## Repository layout

This is a multi-module repo:
- `.` — the core `checker` module (`go.mod`, go 1.23.2). No external dependencies.
- `gin/` — `github.com/cinar/checker/v2/gin`, its own `go.mod`/`taskfile.yml`, `replace`s the core module with `../`.
- `echo/` — `github.com/cinar/checker/v2/echo`, same pattern as `gin/`.
- `checkerlint/` — `github.com/cinar/checker/v2/checkerlint`, a `go/analysis` static analyzer for `checkers`/
  `validate` struct tags, plus its `cmd/checkerlint` standalone binary. Same `replace ../` pattern as `gin`/`echo`,
  but pins newer linter versions in its own `taskfile.yml` (it needs a newer Go toolchain than the core module's
  `go 1.23.2` for its `golang.org/x/tools` dependency). Its `testdata/src/...` tree stands in for the real
  `github.com/cinar/checker/v2` import path so `analysistest` fixtures can exercise custom-checker detection
  without pulling in the real module — don't confuse it with an actual dependency.
- `cli/` — `github.com/cinar/checker/v2/cli`, a standalone command-line interface (`cmd/checker` binary) that runs
  any registered checker/normalizer via `CheckWithConfig` against a value from a shell script, CI pipeline, or Git
  hook. Same `replace ../` pattern as `gin`/`echo`/`checkerlint`; has no external dependencies at all (no `go.sum`),
  since it only depends on the core module.
- `nethttp/` — `github.com/cinar/checker/v2/nethttp`, a `net/http` adapter (`Bind`/`Check`) built only on
  `encoding/json` + `net/http`. Same `replace ../` pattern as `gin`/`echo`; no external dependencies at all (no
  `go.sum`), same as `cli/`.
- `fiber/` — `github.com/cinar/checker/v2/fiber`, a Fiber v3 adapter (`Bind`/`Check`). Same `replace ../` pattern
  as `gin`/`echo`.
- `nfkc/` — `github.com/cinar/checker/v2/nfkc`, registers an `nfkc` checkers-tag normalizer (Unicode
  Normalization Form KC) via `checker.RegisterMaker` in its `init` function — a blank import is enough to use
  it. Its own module, not part of the core, because NFKC needs `golang.org/x/text/unicode/norm`; keeping it
  isolated keeps the core module's zero-dependency promise intact for callers who don't opt in. Same
  `replace ../` pattern as the other adapters.
- `checkergen/` — `github.com/cinar/checker/v2/checkergen`, generates a `Check<Type>(v *Type)
  (checker.CheckErrors, bool)` function per eligible struct, calling the same checker/normalizer plain
  functions (`checker.IsEmail`, `checker.MinLen[string](8)`, ...) directly via `checker.Check` instead of
  walking the struct with `reflect` at runtime — coexists with `CheckStruct`, not a replacement for it. Same
  `replace ../` pattern as `checkerlint`, and needs the same newer Go toolchain for `golang.org/x/tools`
  (`go/packages`, for type info). Its `testdata/fixture/` and `testdata/ineligible/` hold committed,
  regenerated-by-test `checkergen_generated.go` fixtures (see `generate_test.go`'s
  `TestGenerateMatchesCommittedFixture` for the drift check, and `differential_test.go` for the
  behavioral-parity-with-`CheckStruct` check) — regenerate them with
  `go run ./cmd/checkergen ./testdata/<dir>` after a `callSpecs` change, same as any other generated-and-committed
  code. A test that calls `Generate` with anything other than a plain unfiltered rerun (e.g. a `-type` filter)
  must target a scratch copy (`copyToScratch` in `generate_test.go`), never a real `testdata` package directly,
  since `Generate` overwrites that package's committed, shared output file as a side effect.
- `locales/` — the `locales` package, one file per locale (e.g. `en_us.go`) plus `locales.go` and `locales_test.go`.

Each module has its own `taskfile.yml` and is built/linted/tested independently (see `.github/workflows/ci.yml`,
which runs parallel jobs per module: root, `gin/`, `echo/`, `nethttp/`, `fiber/`, `nfkc/`, `checkerlint/`,
`checkergen/`, `cli/`).

## Commands

All commands use [Task](https://taskfile.dev) (`go run github.com/go-task/task/v3/cmd/task@v3.38.0`), run from
the module's own directory (root, `gin/`, `echo/`, `nethttp/`, `fiber/`, `nfkc/`, `checkerlint/`, `checkergen/`,
or `cli/`):

- `task` (default) — runs `fmt`, `lint`, then `test`, in that order.
- `task fmt` — `go fix ./...`
- `task lint` — `go vet`, `gosec` (excludes
  `gin`/`echo`/`examples`/`checkerlint`/`cli`/`nethttp`/`fiber`/`nfkc`/`checkergen` dirs when run from root —
  `gosec` doesn't respect Go module boundaries or the `testdata` convention the way `go build`/`go vet`/`go test`
  do, so any subdirectory with its own `go.mod`, or a `testdata` tree with fixtures that don't type-check

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cinar/checker](https://github.com/cinar/checker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
