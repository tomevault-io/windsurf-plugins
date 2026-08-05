---
trigger: always_on
description: Guide for AI agents working in the `carapace-spec` repository.
---

# AGENTS.md

Guide for AI agents working in the `carapace-spec` repository.

## Project Overview

`carapace-spec` defines shell completions using YAML spec files, built on top of [carapace](https://github.com/carapace-sh/carapace). A spec file describes a command's flags, subcommands, and completion actions; the library converts that into a `cobra.Command` tree that carapace bridges to all supported shells (bash, zsh, fish, elvish, nushell, oil, powershell, tcsh, xonsh).

The `carapace-spec` binary loads a spec YAML and outputs shell-specific completion scripts. [carapace-bin](https://github.com/carapace-sh/carapace-bin) is the recommended runtime for end users (it ships many custom macros), but this repo provides the library, spec format, codegen, and a standalone binary.

## Essential Commands

```bash
go build -v ./...                              # build all packages
go test -v ./...                               # run all tests
go test -v -coverprofile=profile.cov ./...     # tests with coverage (CI uses this)
gofmt -d -s .                                  # check formatting (CI fails on any diff)
go vet ./...                                   # vet
staticcheck ./...                              # static analysis (CI runs this)
go generate ./...                              # regenerate schema.json from struct tags
mdbook build docs                              # build the documentation site
```

### Schema generation

`schema.json` is generated from Go struct tags via `cmd/schema/main.go`:

```bash
go generate ./...          # runs: go run -C cmd/schema . ../../schema.json
```

### Releasing

Releases are handled by GoReleaser (`.goreleaser.yml`), triggered by git tags. Builds target linux/windows/darwin (CGO disabled) plus termux/android (CGO enabled). Packages are published to Homebrew, Scoop, AUR, and nFPM (apk/deb/rpm). Agents should not trigger releases.

## Repository Structure

This is a **Go workspace** (`go.work`) with two modules:

| Module | Path | Purpose |
|--------|------|---------|
| Library | `.` (`github.com/carapace-sh/carapace-spec`) | Spec types, YAML→cobra conversion, macro engine, codegen |
| CLI | `./cmd` (`github.com/carapace-sh/carapace-spec/cmd`) | The `carapace-spec` binary, schema generator |

The `cmd` module replaces `github.com/carapace-sh/carapace-spec` with the local library via a `replace` directive in `cmd/go.mod`.

### Library package layout (root module, package `spec`)

- `command.go` — `Command` type alias + `ToCobra()`/`ToCobraE()`: converts a spec `Command` into a `*cobra.Command`, wiring flags, completions, subcommands, run handlers
- `flag.go` — `addFlagTo()`: maps spec `Flag` to the correct `pflag.FlagSet` method (Bool/String/Count, P/N/S variants)
- `fork.go` — `flagSet` wrapper that detects whether the pflag fork (`carapace-pflag`) is in use via reflection (`IsFork()` checks for `BoolN` method)
- `action.go` — `action`/`value` types, `ActionMacro()`, `ActionSpec()`, and `action.Parse()` which resolves macro strings and `|||` modifier chains into `carapace.Action`s
- `modifier.go` — `modifier.Parse()`: applies modifier macros (`$filter`, `$list`, `$chdir`, etc.) to an action
- `macro.go` — `Macro` struct, `MacroN`/`MacroI`/`MacroV` constructors, `AddMacro`/`AddMacroI`/`AddMacroV` for custom macros, `macroName()` auto-naming logic
- `macromap.go` — `MacroMap` type + `Format()` method that generates Go source for a macro map (used by carapace-bin)
- `core.go` — `init()` registers core macros (`$files`, `$directories`, `$executables`, `$spec`, shell macros like `$bash`, `$sh`, etc.)
- `register.go` — `Register()`: adds the `_carapace macro` subcommand to a cobra command for macro introspection/invocation
- `run.go` — `run` type with three run modes: `parseMacro()`, `parseScript()`, `parseAlias()`
- `scan.go` — `ScanMacros()`: scans Go packages via `go doc` to discover `Action*` functions for macro registration (experimental)
- `codegen.go` — `Codegen()`: generates Go source files (cobra commands) from a spec, writing to a temp directory
- `schema.go` — embeds `schema.json` via `//go:embed`, exposes `Schema()`
- `internal/pflagfork/flag.go` — introspects pflag `Flag` fields (Nargs, Mode, OptargDelimiter) via reflection; classifies flags as Default/ShorthandOnly/NameAsShorthand
- `internal/shebang/shebang.go` — parses `#!` shebang lines for script run mode
- `pkg/command/` — `Command`, `Flag`, `FlagSet`, `Run`, `Parsing` types with YAML (de)serialization
- `pkg/macro/macro.go` — generic `Macro[T]`, `MacroMap[T]`, `MacroN`/`MacroI`/`MacroV` with YAML arg parsing and signature generation

### CLI package layout (`cmd/` module)

- `cmd/carapace-spec/main.go` — entry point, version handling
- `cmd/carapace-spec/cmd/root.go` — root command; loads spec YAML, bridges completion output. Contains `loadSpec()` and `bridgeCompletion()` (rewrites `_carapace` callbacks to reference the spec file path)
- `cmd/carapace-spec/cmd/codegen.go` — `codegen` subcommand
- `cmd/carapace-spec/cmd/run.go` — `run` subcommand (executes a spec as a real command)
- `cmd/carapace-spec/cmd/selfupdate.go` — self-update via `carapace-selfupdate`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carapace-sh/carapace-spec](https://github.com/carapace-sh/carapace-spec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
