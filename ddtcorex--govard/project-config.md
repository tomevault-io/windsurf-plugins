---
trigger: always_on
description: This document is the project-specific operating manual for AI coding agents working in `govard`.
---

# AGENTS.md

This document is the project-specific operating manual for AI coding agents working in `govard`.

## 1. Mission and Product Context

`Govard` is a Go-based local development orchestrator for PHP and web projects (Magento, Laravel, Symfony, WordPress, etc.), with:

- CLI orchestration (`govard ...`)
- Container runtime automation (Docker)
- Blueprint/template rendering
- Remote environment tooling
- SSL/proxy utilities
- Wails-based Desktop GUI Application (`govard desktop`)

Primary goals for contributions:

1. Preserve CLI stability and predictable behavior.
2. Keep workflows fast for local developers.
3. Avoid regressions in core command families (bootstrap/proxy/db/sync/remote/lock/domain/desktop).
4. Maintain UI reliability and backend bridge logic for the Desktop App.
5. Maintain release quality (GoReleaser, checksums, install paths).

## 2. Runtime and Toolchain Requirements

- Go: `1.25+` (module uses `go 1.25.0`)
- Node.js: `20` (for frontend UI and frontend tests)
- Wails: `v2.11+` (for desktop app development)
- Docker: required for integration tests and runtime orchestration
- GitHub CLI (`gh`): useful for release inspection (optional)

Local sanity checks:

```bash
go version # If not in PATH, check /home/$USER/go_dist/go/bin/go
node --version
wails version
docker --version
```

## 3. Repository Map

- `cmd/govard/main.go`: CLI entrypoint
- `cmd/govard-desktop/`: Desktop app entrypoint (built by Wails)
- `desktop/`: Wails desktop app codebase (Go backend bridge + HTML/vanilla JS frontend in `desktop/frontend/`)
- `wiki/`: Project documentation (GitHub Wiki source files, synced automatically on push)
- `internal/cmd/`: Cobra command implementations
- `internal/engine/`: orchestration, config, blueprint logic
- `internal/engine/bootstrap/`: framework bootstrap workflows
- `internal/engine/remote/`: remote sync/deploy/ssh helpers
- `internal/proxy/`: caddy/proxy route and TLS helpers
- `internal/updater/`: update-check notification logic
- `internal/ui/`: terminal rendering helpers
- `tests/`: unit/contract tests (default location for tests)
- `tests/integration/`: integration tests (tagged and heavier)
- `tests/frontend/`: Node test runner suite for frontend pieces
- `install.sh`: unified installer (release binary by default, `--source` for source build)
- `scripts/build-macos-pkg.sh`: helper script to build macOS `.pkg` installer artifacts
- `.goreleaser.yml`: release artifact config
- `.github/workflows/`: CI/release/security automation

## 4. Core Build and Test Commands

Preferred commands:

```bash
make test                # lint + fmt-check + vet + frontend + unit + integration tests
make test-unit           # unit tests only
make test-integration    # integration tests (requires build + docker)

make vet                 # go vet
make fmt                 # go fmt ./...
make build               # build Govard for the current platform
```

Useful direct commands:

```bash
go test ./...
go test ./tests/... -v
go test -tags integration ./tests/integration/... -v -timeout 30m
```

## 5. CI and Quality Gates

Key workflows:

- `ci-pipeline.yml`: vet + gofmt check + fast tests + integration + binary build
- `release.yml`: triggered by tag `v*.*.*`, runs GoReleaser and uploads macOS `.pkg` installers
- `codeql.yml`: code scanning
- `govulncheck.yml`: weekly vulnerability scan

Do not assume local success if you skipped:

1. formatting (`gofmt -s -w` / `make fmt`)
2. tests relevant to changed surface
3. at least one command-level smoke check for CLI behavior changes

## 6. Testing Conventions (Important)

Project convention is to keep most tests in `tests/` package `tests`.

Test isolation and fixture hygiene rules:

- Keep tests hermetic: do not rely on the user's real projects, real containers, or machine-specific state.
- Do not use real/legacy project fixture names (for example `magento2-test-instance`); use neutral fixtures such as `sample-project`.
- Prefer mocks/fakes over live network dependencies in unit tests (for HTTP, inject client/transport and use a mock `RoundTripper`).
- Ensure test suites isolate state via temporary `GOVARD_HOME_DIR` (use `TestMain` setup where appropriate).
- If an integration test must touch external services, gate it with explicit env checks and clear skip reasons.

When you need to test internal logic from `internal/cmd` (or other internal packages):

1. keep production helpers unexported where possible
2. add narrow exported wrappers suffixed with `ForTest` (example: `BuildLocalDBResetScriptForTest`)
3. consume those wrappers from test files in `tests/`

Example pattern:

- production function: `buildThing(...)`
- test wrapper: `BuildThingForTest(...)`
- test location: `tests/thing_test.go`

Avoid broad export of internals just for tests.

## 7. CLI Architecture and Command Work

`internal/cmd/root.go` owns root registration and `Version` binding.

When adding/modifying a command:

1. define command in `internal/cmd/<area>.go`
2. register with `rootCmd.AddCommand(...)` (or relevant subcommand group)
3. ensure flags are explicit and help text is actionable
4. return errors with context (`fmt.Errorf("operation: %w", err)`)
5. add/adjust tests in `tests/`
6. update docs for user-visible command/flag changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ddtcorex/govard](https://github.com/ddtcorex/govard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
