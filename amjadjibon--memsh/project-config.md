---
trigger: always_on
description: This file provides guidance to AGENTS when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AGENTS when working with code in this repository.

## Project Overview

`memsh` is a **virtual bash shell** implemented in Go. It executes bash-like commands against an `afero.MemMapFs` in-memory filesystem — the real OS filesystem is never touched. Shell parsing/interpretation is delegated to `mvdan.cc/sh/v3`; all commands (built-ins and extensions) are native Go plugins or WASM plugins.

**Security model:** external OS commands are blocked by default. Only registered plugins can run. Opt-in via `WithAllowExternalCommands(true)`.

## Repository Guidelines (Contributor Quick Guide)

### Project Structure
- `cmd/`: CLI subcommands and Cobra wiring.
- `pkg/shell/`: core runtime, virtual FS integration, and command/plugin execution.
- `pkg/shell/plugins/native/`: built-in command implementations.
- `internal/`: session, server, REPL, config, and agent internals.
- `tests/`: integration tests using in-memory FS helpers.
- `web/`: static web terminal assets.

### Build, Test, and Lint
- `make build` builds `./bin/memsh`.
- `make plugins` compiles WASM plugins.
- `make test` runs `go test ./... -v -count=1`.
- `make cover` writes `coverage.out` and `coverage.html`.
- `make lint` runs `go vet`.

### Style and Naming
- Use `gofmt` formatting; prefer `goimports` for import cleanup.
- Keep package names lowercase; exported identifiers in `CamelCase`.
- Use descriptive, feature-based filenames (for example `snapshot.go`, `server.go`).
- For commands/plugins, keep names lowercase and shell-like (`jq`, `grep`, `mktemp`).

### Tests and PRs
- Add/extend `*_test.go` files near changed code or in `tests/` for command behavior.
- Prefer `tests.NewTestShell(...)` to test commands consistently.
- Follow Conventional Commit prefixes: `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`.
- PRs should include a concise summary, linked issue(s), and test evidence (`make test`).

## Commands

```bash
make build          # build ./bin/memsh
make test           # run all tests
make cover          # tests with coverage
make lint           # vet (go vet ./...)
make clean          # remove binaries and .wasm files
make install        # install to /usr/local/bin

go run .                        # interactive REPL
go run . ./scripts/etl-pipeline.sh  # run a script file
echo "ls /" | go run .          # non-interactive pipe

# Plugin management
go run . plugin list            # list installed plugins
go run . plugin install python  # install Python 3.12 WASM runtime
go run . plugin install ruby    # install Ruby 3.2 WASM runtime
go run . plugin install php     # install PHP 8.2.6 WASM runtime
go run . plugin install /path/to/plugin.wasm  # install local WASM plugin

go test ./...                       # full test suite
go test ./tests -v                  # integration tests verbose
go test ./tests -run TestJq -v      # single test suite
go test ./pkg/shell/... -run TestName  # shell package tests

# HTTP server (sessions always enabled, TTL 30m default)
go run . serve
go run . serve --addr :3000 --session-ttl 1h --cors
```

Test suites in `tests/`: `TestAwk`, `TestBase64`, `TestFind`, `TestGrep`, `TestGo`, `TestGoja`, `TestJq`, `TestLua`, `TestPhp`, `TestPython`, `TestRuby`, `TestWc`, `TestYq`, `TestGit`, `TestSQLite`, `TestSnapshot`, and more.

## Architecture

```
Shell.Run(ctx, script)
  → syntax.NewParser().Parse()           # mvdan.cc/sh parses bash syntax
  → interp.Runner.Run(ctx, ast)          # mvdan.cc/sh interprets AST
        ↓ interp.OpenHandler             # pkg/shell/fs.go: all file I/O → afero.MemMapFs
        ↓ interp.ExecHandlers
              WithShellContext()         # injects FS+cwd+aliases into ctx
              s.builtins[cmd]            # native Go plugins (each in pkg/shell/plugins/native/<cmd>.go)
              s.plugins[cmd]             # WASM plugins via wazero
              blocked (or next())        # external OS commands blocked by default
```

**Key files:**
- `pkg/shell/shell.go` — `Shell` struct, `New()`, `execHandler`, `changeDir`, `sourceFile`, one wazero runtime per shell, WASM pre-compiled at startup. After `Run`, `s.cwd = s.runner.Dir`.
- `pkg/shell/options.go` — all functional options: `WithFS`, `WithCwd`, `WithEnv`, `WithStdIO`, `WithPlugin`, `WithBuiltin`, `WithPluginBytes`, `WithWASMEnabled`, `WithPluginFilter`, `WithDisabledPlugins`, `WithAllowExternalCommands`, `WithInheritEnv`, `WithAliases`.
- `pkg/shell/fs.go` — `openHandler` wires all file I/O to afero; `resolvePath` always returns absolute paths.
- `pkg/shell/plugin.go` — WASM registry; `runWASIPlugin` (`_start` export) vs `runCustomPlugin` (`run` export).
- `pkg/shell/wasi_fs.go` — `aferoSysFS`: implements `experimentalsys.FS` on top of `afero.Fs`, mounted via wazero so WASI modules read/write the virtual FS directly.
- `pkg/shell/defaults.go` — `defaultNativePlugins()` slice and `defaultPlugins` WASM map.
- `pkg/shell/plugins/plugin.go` — `Plugin`, `PluginInfo`, `ShellContext` interfaces; `ShellCtx(ctx)`, `WithShellContext()`.
- `pkg/shell/snapshot.go` — `TakeSnapshot`/`RestoreSnapshot` for serializing `afero.MemMapFs` to JSON.
- `internal/server/server.go` — HTTP handler (`Handler`), all route registrations and request logic.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amjadjibon/memsh](https://github.com/amjadjibon/memsh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
