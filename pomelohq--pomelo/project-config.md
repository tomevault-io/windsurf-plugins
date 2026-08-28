---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What Pomelo is

A native macOS app that spins up a full, isolated, runnable dev environment for **every branch** of a multi-repo project — services, databases, shared infra, each branch a real git worktree with its own ports/DBs. The product **is** the SwiftUI app (`desktop/PomeloApp`); it links the Go core via `libpom` (c-archive FFI) and is **portless** — no dashboard, no HTTP server, no browser UI. There is also a plain `pom` CLI (same core). The old React web UI, `internal/web/`, TUI, and `pom daemon`/`pom update` are all gone.

## Commands

```bash
make build          # go build -o pom ./cmd/pom
make test           # go test ./...
make vet            # go vet ./...
make check          # build + vet + test (the Go gate)
make app            # build the native app (desktop/PomeloApp/build.sh: Go c-archive → SwiftPM link)
make app-run        # build + launch the app
make dmg            # local signed DMG for testing (never publishes)
```

- Single Go test: `go test ./internal/services/ -run TestEnvFileEntries`
- App unit tests: `cd desktop/PomeloApp && xcodebuild test -scheme PomeloApp -destination 'platform=macOS,arch=arm64' -derivedDataPath .ddata -skipPackagePluginValidation` (ViewModels vs `MockPomAPI`; add `-only-testing:PomeloAppTests/<Suite>` to scope). NOTE: `swift test` does NOT work — the CodeEditSymbols dep needs Xcode's resource pipeline. Needs `Vendor/libpom.a` (run `build.sh` once).
- Requires: Go 1.26+, `zsh`, Xcode + `codesign` (macOS, Apple Silicon). No tmux — services/shells run on self-managed PTY holders.

## Release (CI-only)

```bash
make patch          # bump BOTH version consts → commit → tag v<x> → push
make minor / major
```

`make patch/minor/major` bumps `cmd/pom/root.go` `version` **and** `cmd/libpom/libpom.go` `appVersion` in lockstep (`make version-check` guards drift). Pushing the `v*` tag is the entire release: `.github/workflows/release.yml` → `app-build.yml` (`publish:true`) builds → signs → notarizes → DMG → Sparkle appcast → GitHub Release. **No local publish path.** The app self-updates via Sparkle. Full details + required CI secrets in `RELEASE.md`. Never delete old releases; never regenerate the Sparkle EdDSA key.

Before cutting a release, run the `release-audit` skill (`.claude/skills/release-audit`): CI builds the GitHub Release notes and the Sparkle appcast from the `## [<version>]` block in `CHANGELOG.md`, so that block MUST be curated and committed before the tag is pushed — otherwise notes fall back to an auto PR list.

## Architecture (big picture)

**One Go core, two front doors.** `internal/core` holds the `Server` + business logic. It is reached two ways, never over HTTP:

- **Native app** (`desktop/PomeloApp`, SwiftUI) → `libpom` FFI → `internal/core` → `internal/services` / `internal/pipeline` / `internal/ptyhost`.
  - Reads/actions: typed per-domain bindings in `cmd/libpom/bindings.go` (`//export Pom<Domain>` → a `Server.<Method>`) → Swift `PomCore.<domain>Data()`. Adding an endpoint = extract a `Server` data method + one `//export` + one `PomCore` method. There is **no generic `/api` bridge** — the app must not feel client-server.
  - Streams (PTY / Claude / pipeline) → direct C-FFI (`cmd/libpom/stream.go` `PomStream*`).
- **CLI**: `cmd/pom/main.go` (dispatch only) → `internal/services` → `internal/ptyhost`.
- **MCP**: `pom mcp` is a portless stdio MCP server (hand-rolled JSON-RPC) exposing a workspace's env to agents (ports/db/services/run_in_env/config/doctor). It builds its handler in-process from the `pom.yml` found by walking up from CWD — no running app needed. Auto-registered into Claude windows. Agent-facing endpoints: `internal/core/mcp_endpoints.go`.

The app re-execs its **own bundle binary** for the `pty` / `mcp` / `prepare-main` / `claude-hook` subcommands (`pombin.Path() = os.Executable()`), so it needs no external `pom` installed.

### Go layout

```
cmd/pom/main.go        — CLI entry, dispatch only (no business logic)
cmd/pom/cmd_*.go       — one cobra command per file (start, workspace, db, run, mcp, onboard, …)
cmd/libpom/            — c-archive FFI: bindings.go (typed reads/actions), stream.go (PomStream*), libpom.go (appVersion)
internal/
  core/                — the Server + all business logic (feature file per domain; mcp_endpoints.go, onboard.go, activity.go, control.go, run_actions.go, …)
  services/            — infra layer / side effects: env resolution, port/slot allocation, docker, git, files, holder spawn + the Holder interface
  ptyhost/             — self-managed PTY holders (tmux-free): a detached process per service/shell behind a Unix socket, ring-buffer scrollback, multi-client attach/resize
  pipeline/            — staged workspace create/delete (runner + Event channel)
  config/              — pom.yml parse, dot-notation template resolution, load-time validation; pom.d/**/*.yml deep-merge
  provider/            — shell (zsh variants), tracker (jira), forge (gh/git host), dbclient
  agent/               — built-in Claude Code agent registry + hooks; agent/claude the managed CLI launcher

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pomelohq/pomelo](https://github.com/pomelohq/pomelo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
