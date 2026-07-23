---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

`zcli` is the Zerops CLI written in Go (module `github.com/zeropsio/zcli`, Go 1.26). The entrypoint is `cmd/zcli/main.go`, which delegates to `src/cmd.ExecuteCmd()`. All implementation lives under `src/`.

## Common commands

- `make test` — runs `go test -v ./cmd/... ./src/...`. Run a single test with `go test -v -run TestName ./src/<pkg>/...`.
- `make lint` — runs the pinned `./bin/golangci-lint` for darwin/arm64, linux/amd64, and windows/amd64. Config: `.golangci.yaml`.
- `make build-dev` — build a dev binary for the host into `./bin/zcli` (debug-friendly via `-gcflags='all=-l -N'`, version injected from git).
- `make all` — cross-builds dev binaries for windows-amd, linux-amd, darwin-amd, darwin-arm. Single-target builds: `make linux-amd` etc. All build targets share the `DEV_BUILD` recipe in the Makefile (version metadata, gcflags).
- `make install` — production install into `~/.local/bin/zcli` (same path as the public `install.sh` script). Stripped/optimized, version from `git describe`, same flag set as `.goreleaser.yaml`.
- `make install-dev` — installs to `$GOBIN` (or `$GOPATH/bin`) as `zcli-dev` so it sits with other Go dev tools and coexists on PATH with a production `zcli`.
- `make goreleaser-check` / `make goreleaser-snapshot` — validate `.goreleaser.yaml` and dry-run a release build to `./dist`.
- `make tools` — install pinned dev tooling (`golangci-lint`, `goreleaser`) into `./bin`. Versions are declared at the top of the Makefile; bumping them triggers reinstall via stamp files. `make lint` and the goreleaser targets depend on `tools`, so first invocation auto-installs.
- `make showcase` — runs `src/uxBlock/showcase/main.go` to preview UI elements.

## Architecture

### Command layer (`src/cmd`, `src/cmdBuilder`)

Commands are not declared directly against Cobra. Instead, `src/cmdBuilder` wraps Cobra with a fluent builder (`cmdBuilder.NewCmd().Use(...).Arg(...).StringFlag(...).LoggedUserRunFunc(...)`) and `buildCobraCmd.go` translates it into a `*cobra.Command`. The root command lives in `src/cmd/root.go` and registers all subcommands via `AddChildrenCmd`.

Two run-function flavors are central to how commands behave:

- `GuestRunFunc(func(ctx, *GuestCmdData) error)` — runs when there is no auth token. Has access to `CliStorage`, `UxBlocks`, parsed `Args`/`Params`, `Stdout`/`Stderr` printers.
- `LoggedUserRunFunc(func(ctx, *LoggedUserCmdData) error)` — runs when a token is present (from `cliStorage` or the `CliTokenEnvVar` env var). Embeds `*GuestCmdData` and adds `RestApiClient` (a `zeropsRestApiClient.Handler` pointed at the configured region/host), plus optional resolved `Project`/`Service` entities and a `ProjectSelector`.

If only `GuestRunFunc` is defined, it runs for logged-in users too. If only `LoggedUserRunFunc` is defined and no token is set, the command exits with `UnauthenticatedUser`. Implementation: `src/cmdBuilder/createRunFunc.go`.

### Scope levels (`src/cmdBuilder/scopeProject.go`, `scopeService.go`)

Commands that operate on a project or service set `.ScopeLevel(cmdBuilder.ScopeProject(...))` or `.ScopeLevel(cmdBuilder.ScopeService(...))`. The scope level adds the relevant flags/args and, before the run function executes, resolves the target by ID/name into `LoggedUserCmdData.Project` / `.Service` — possibly creating a new project/service interactively when `WithCreateNewProject()` / `WithCreateNewService()` are passed (see `serviceDeploy.go` for an example). Persisted "scope" (saved project) is read from `CliStorage.Data().ScopeProjectId`.

### Flag normalization

`ExecuteRootCmd` sets a Cobra normalization function that converts camelCase flag names to kebab-case (`src/cmdBuilder/executeRootCmd.go`). Flags can be defined as camelCase in code and used as kebab-case on the CLI.

### Persistent state and config

- `cliStorage` (`src/cliStorage`) wraps `storage.New[Data]` (`src/storage`) — a JSON-backed file used to persist the login token, region selection, saved project scope, and VPN key registry. Path comes from `constants.CliDataFilePath()`.
- `constants` exposes platform-specific paths, env var names (e.g. `CliTokenEnvVar`), and the default region (used when no region is configured).
- Log file path comes from `constants.LogFilePath()`; loggers (`src/logger`) split human output from a debug file logger that captures `LogDebug`/error traces.

### UX layer (`src/uxBlock`, `src/uxHelpers`, `src/printer`, `src/terminal`)

All terminal output and interactive prompts go through `uxBlock.Blocks` (constructed in `executeRootCmd.go`). It owns the loggers, terminal size, an `isTerminal` flag, and a cancel function wired to SIGINT/SIGTERM. `uxBlock/models` provides Bubble Tea TUI models (selectors, spinners, tables); `uxHelpers` wraps common flows like `ProjectSelector`. `printer` is the lower-level line printer used directly by commands (`cmdData.Stdout.PrintLines(...)`). `styles` centralizes lipgloss styles, including the custom Cobra help template defined in `src/cmd/root.go`.

### API client and entities


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeropsio/zcli](https://github.com/zeropsio/zcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
