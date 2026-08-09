---
trigger: always_on
description: - On Windows, prefer PowerShell with `pwsh -NoProfile` for scripts and commands. On Linux and macOS, use the available native shell.
---

# AGENTS.md

## Working agreements

- On Windows, prefer PowerShell with `pwsh -NoProfile` for scripts and commands. On Linux and macOS, use the available native shell.
- Prefer `pnpm` if Node.js tooling is introduced. Ask before adding any new production dependency.
- Keep reusable device and OS configuration scripts in `~/Workspace/dotfiles-and-scripts`. Keep generated reports, logs, state, caches, and other runtime output outside that repository.
- The project evolves continuously. When a change materially alters repository structure, workflows, compatibility assumptions, or other durable facts, update `AGENTS.md` as part of the same work so it remains accurate; keep additions concise and avoid documenting temporary implementation details.
- The project primarily serves Chinese-speaking users, so user-facing UI text should be in Chinese. Keep it concise and use English only where useful.
- Developer-facing content may use English, including internal output, logs, daemon output, configuration language, and code comments.

## Project overview

`kd` is a small, cross-platform command-line dictionary distributed as a single binary. It looks up words and long text, uses a local SQLite cache, and delegates online queries to a localhost daemon. It also contains optional TTS, self-update, paging, logging, and shell-completion features.

- Module: `github.com/Karmenzind/kd`
- Go version: 1.26.5 (see `go.mod`)
- CLI entry point: `cmd/kd/kd.go`
- CLI framework: `github.com/urfave/cli/v3`; construct a fresh root `cli.Command` for tests and keep framework types inside `cmd/kd`.
- Auxiliary TTS test program: `cmd/ttstest/main.go`
- The SQLite driver is the pure-Go `modernc.org/sqlite`; builds and cross-builds must keep `CGO_ENABLED=0` and require no C compiler.

## Design philosophy

- Prefer simplicity over flexibility, readable direct code over abstraction, and long-term maintenance cost over theoretical extensibility.
- Use the standard library first. Add a dependency only for a clear technical reason that outweighs its ongoing update, security, binary-size, and compatibility costs.
- Do not turn `kd` into a general platform. Avoid frameworks, plugin systems, dependency-injection machinery, and layers that do not solve a current concrete problem.
- Add configuration, persistent state, top-level commands, and background behavior sparingly because each becomes a compatibility and maintenance obligation. Do not add another background service alongside the existing narrowly scoped daemon.
- When several implementations are reasonable, choose the one with fewer concepts, less state, and lower long-term maintenance cost. Do not add a dependency merely to save a small amount of straightforward code.
- For this interactive CLI, prioritize startup latency, response time, and memory use. Measure before optimizing; do not trade away readability for unsupported micro-optimizations.

## Repository map

- `cmd/kd`: urfave/cli v3 root command, flags, startup, configuration wiring, and result presentation.
- `config`: TOML configuration defaults, loading, validation, and sample output.
- `internal/query.go` and `internal/server.go`: CLI-side query orchestration and the thin server entry point retained for compatibility.
- `internal/query`: cache and online query handling, Youdao response parsing, and output formatting.
- `internal/ui`: delayed, TTY-only query progress rendering and terminal capability fallbacks; it must not contain query business logic.
- `internal/cache`: SQLite data, not-found data, and query counters.
- `internal/daemon`: daemon lifecycle, TCP client/server communication, runtime state, process discovery, and scheduled maintenance.
- `internal/model`: shared request, response, result, and runtime data structures.
- `internal/run`: runtime metadata, cache paths, and the fixed daemon port (`19707`).
- `internal/tts` and `internal/update`: optional speech and binary-update flows.
- `pkg`: focused filesystem, HTTP, terminal, process, string, and decoration helpers.
- `scripts/build.sh`: release build logic used by GitHub Actions for Linux amd64/arm64, macOS amd64/arm64, and Windows amd64.

## Development workflow

Use repository-root package paths such as `./cmd/kd` for local builds.

```shell
go test -count=1 ./...
go vet ./...
gofmt -w path/to/changed.go
```

Build with `CGO_ENABLED=0`. On Unix use `CGO_ENABLED=0 go build -o build/kd ./cmd/kd`; in PowerShell set `$env:CGO_ENABLED = "0"` before running the equivalent `go build` command.

Before handing off a Go change, run `gofmt` on changed files and at least `go test ./...`. Run `go vet ./...` for changes beyond documentation. After a substantial change is pushed, inspect the GitHub Actions CI result across Linux, macOS, and Windows; investigate failures instead of assuming local success covers platform-specific behavior. The checked-in `Makefile` and release script assume Unix tools; prefer direct Go commands during cross-platform development on Windows.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Karmenzind/kd](https://github.com/Karmenzind/kd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
