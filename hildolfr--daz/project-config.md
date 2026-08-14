---
trigger: always_on
description: - Daz is a Go-based modular Cytube bot.
---

# AGENTS.md

## Repository Overview
- Daz is a Go-based modular Cytube bot.
- Source is organized under `cmd/`, `internal/`, and `pkg/`.
- Plugins live under `internal/plugins/`.
- Build output is `./bin/daz`.

## Build / Run
- Build: `make build` (runs `./scripts/build-daz.sh`).
- Run: `make run` (depends on `make build`).
- Direct run: `./bin/daz` after build.
- Config: copy `config.json.example` to `config.json`.

## Lint / Format
- Format: `make fmt` (runs `go fmt ./...`).
- Lint: `make lint` (runs `go vet ./...`).
- Manual gofmt: `gofmt -w <file.go>` when editing a file.

## Tests
- All tests: `make test` (runs `go test -race ./...`).
- Single package: `go test ./internal/plugins/greeter`.
- Single test name: `go test ./internal/plugins/greeter -run TestGetGreeting`.
- Single test with fresh cache: `go test ./internal/plugins/greeter -run TestGetGreeting -count=1`.
- Run a test across all packages: `go test ./... -run TestName`.
- Useful env: set `LOG_LEVEL=debug` to enable verbose logging.
- Live Cytube admin-plane/manual verification room: `daztest` (`https://cytu.be/r/daztest`).
- When a live test against a chat user is needed, use `daztester` in `daztest` rather than the production room.
- Default all future operator/admin-plane testing maneuvers to `daztest`; avoid using `always_always_sunny` for routine verification unless explicitly required.

## External Agent Rules
- No `.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md` found.
- For this repo, push with the GitHub identity `hildolfr` while dual-account work is active; do not accidentally push Daz changes as `segordon`.

## Project Decision Log
- 2026-06-30: For this project, chat is not currently treated as private/confidential. The requester explicitly stated that chat log leakage/visibility is acceptable unless requirements change. This applies to the `!bug` chat-reporting flow and its captured chat context (see `internal/plugins/commands/bug/plugin.go` and the deploy verification note in `docs/deploy-checklist.md`). No implementation change is requested solely to add chat-log privacy protections based on the current request. If future work handles privacy-sensitive content, revisit this assumption before treating chat or chat-derived logs/issues as confidential.

## Go Version
- Go version in `go.mod`: `go 1.25.9`.
- Toolchain: `go1.25.9` (if you use `GOTOOLCHAIN=auto`).

## Runtime Flags
- `-config` sets config file path (default `config.json`).
- `-health-host` controls the health/metrics bind host (default `127.0.0.1`).
- `-health-port` controls health server port.
- `-verbose` enables debug logging.
- Environment `FORCE_COLOR=1` forces colored logs.

## Imports
- Use standard Go import grouping (stdlib, third-party, local).
- Keep module-local imports prefixed with `github.com/hildolfr/daz/...`.
- Avoid import aliasing unless required to resolve conflicts.

## Formatting
- Always run `gofmt` on touched files.
- Keep lines reasonably short; wrap long literals or comments.
- Use tabs for indentation as produced by `gofmt`.

## Naming
- Exported identifiers use `CamelCase`.
- Unexported identifiers use `lowerCamelCase`.
- Package names are short, lowercase, no underscores.
- Use descriptive names; single-letter names only for trivial loops.
- Constants follow Go conventions; avoid screaming snake-case.

## Types and Structs
- Prefer explicit struct fields over anonymous structs.
- Use pointers for mutable/shared state.
- Keep config structs tagged with JSON tags.
- Use typed aliases only when they add clarity.

## Error Handling
- Return errors instead of panicking.
- Wrap errors with context: `fmt.Errorf("context: %w", err)`.
- In `main`, log errors and `os.Exit(1)` for fatal failures.
- For deferred cleanup, log close errors without overriding main errors.

## Logging
- Use `internal/logger` for app logging.
- Pass plugin/component name as the first argument.
- Use `logger.Debug` for verbose details and `logger.Info` for routine info.
- Avoid `log.Printf` except in CLI utilities that already use it.

## Concurrency
- Protect shared mutable state with `sync.Mutex` / `sync.RWMutex`.
- Use `context.Context` for cancellation of goroutines.
- Ensure goroutines exit on context cancellation or channel close.
- Prefer `sync.WaitGroup` for coordinated shutdown.

## Event Bus & Plugins
- Implement `framework.Plugin` interface for plugins.
- Register plugins via `framework.PluginManager`.
- Subscribe to event bus topics with `eventBus.Subscribe`.
- Use `eventbus` constants for core Cytube events.
- Keep plugin names stable; they are used in logs and routing.

### Command Plugin Contract (EventFilter)
- `command.register` must include a comma-separated `commands` list and stable `req.From` plugin name; EventFilter routes execution to `command.<plugin-name>.execute`.
- Always subscribe the plugin to `command.<plugin-name>.execute` before (or at least in the same startup phase as) command registration.
- Keep registration aliases and subscribed handlers in sync; if you register `sign`, `spin`, etc., verify they resolve to your plugin and execute successfully.
- Prefer one canonical command entry that matches plugin identity (for example `couchcoins` for plugin `couchcoins`) and keep legacy aliases additive.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hildolfr/daz](https://github.com/hildolfr/daz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
