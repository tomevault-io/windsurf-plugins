---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is gottp?

A Postman/Insomnia-like TUI API client built in Go with Bubble Tea. Three-panel layout (sidebar, editor, response) with vim-style modal editing, collections stored as YAML, and 8+ theme support. Supports HTTP, GraphQL (including subscriptions), WebSocket, and gRPC (including streaming) protocols with environment variable interpolation, 7 auth methods (basic/bearer/apikey/oauth2/awsv4/digest/none), request history (SQLite), cURL/HAR/Postman/Insomnia/OpenAPI import/export, response diffing (line + word-level), pre/post-request JavaScript scripting, code generation (8 languages), request chaining/workflows, mock server, and a headless CLI runner.

## Build & Test Commands

```bash
make build            # Build to bin/gottp (with version ldflags)
make run              # Build and run
make test             # go test ./...
make test-race        # go test -race ./...
make test-cover       # Coverage report with atomic mode
make lint             # golangci-lint run
make bench            # Benchmarks for HTTP client, scripting, diff, collection loader
make fuzz             # Fuzz all import parsers (30s each)
make vulncheck        # govulncheck ./...
make install          # go install to $GOPATH/bin
make release-dry-run  # GoReleaser snapshot (test release build)

# Single test / package
go test ./internal/protocol/http/ -run TestClient_GET
go test ./internal/runner/ -v
go test ./internal/auth/digest/
go test ./internal/mock/
```

Launch TUI: `./bin/gottp --collection path/to/file.gottp.yaml`

Headless CLI: `./bin/gottp run collection.gottp.yaml --env Production --output json`

Mock server: `./bin/gottp mock collection.gottp.yaml --port 8080`

CLI subcommands: `run`, `init`, `validate`, `fmt`, `import`, `export`, `mock`, `completion`, `version`, `help`

Environment files: place `environments.yaml` next to the collection file. The first environment is auto-selected on startup.

## Architecture

**Bubble Tea MVU pattern**: All UI components implement `Update(msg) -> (Model, Cmd)` and `View() -> string`. The root model is `internal/app/app.go` which orchestrates panels, overlays, and message routing.

### Critical: Message Types Package

`internal/ui/msgs/msgs.go` is a **shared message types package** created to avoid import cycles between `app` and UI components. All `tea.Msg` types and enums (`AppMode`, `PanelFocus`) live here. Both `app` and all UI packages import `msgs` — never import `app` from UI packages. When adding a new message type, add it to `msgs.go`, not to `app` or UI packages.

### Key Packages

| Package | Role |
|---------|------|
| `internal/app/` | Root model, split into focused sub-modules (see below) |
| `internal/ui/msgs/` | Shared message types (breaks import cycles) |
| `internal/ui/panels/{sidebar,editor,response}/` | Three main panels |
| `internal/ui/components/` | Reusable: KVTable, TabBar, StatusBar, CommandPalette, Help, Modal, Toast, JumpOverlay |
| `internal/ui/theme/` | Theme catalog, lipgloss styles, custom YAML theme loader |
| `internal/ui/layout/` | Responsive three-panel layout calculator |
| `internal/protocol/` | Protocol interface, Registry, HTTP/GraphQL/WebSocket/gRPC clients |
| `internal/core/collection/` | YAML collection model, loader, saver |
| `internal/core/environment/` | Environment variables, `{{var}}` interpolation via `Resolve()`, AES-256-GCM encryption |
| `internal/core/{history,state,cookies,tls}/` | SQLite history, central state, cookie jar, mTLS config |
| `internal/export/` | curl/HAR/Postman/Insomnia export + `codegen/` (8 languages) |
| `internal/import/` | Format auto-detection + curl/Postman/Insomnia/OpenAPI/HAR importers |
| `internal/runner/` | Headless CLI runner, perf baselines, workflow execution |
| `internal/auth/{oauth2,awsv4,digest}/` | Auth implementations |
| `internal/diff/` | Myers diff (line + word-level via `DiffLinesWithWords()`) |
| `internal/scripting/` | JavaScript scripting via goja engine |
| `internal/mock/` | Mock HTTP server with CORS, latency/error simulation |
| `internal/templates/` | Pre-built request templates |
| `internal/config/` | App config from `~/.config/gottp/config.yaml` |

### app.go Sub-Module Structure

The `internal/app/` package is split into focused files:

| File | Contents |
|------|----------|
| `app.go` | `App` struct, `New()`, `Init()`, `Update()`, `View()`, `resizePanels()` |
| `app_keys.go` | `handleGlobalKey()`, `handlePanelKey()`, `updateEditorInsert()`, `cycleFocus()`, `updateFocus()` |
| `app_request.go` | `sendRequest()`, `handleRequestSent()`, `initiateOAuth2()`, introspection/reflection handlers |
| `app_overlays.go` | `handleSwitchTheme()`, `handleImportFile()`, `handleSetBaseline()`, `openExternalEditor()` |
| `app_tabs.go` | `syncTabs()`, `loadActiveRequest()`, `loadHistory()`, `handleRequestSelected()` |
| `app_save.go` | `saveCollection()`, `copyAsCurl()`, `importCurl()`, `handleGenerateCode()`, `handleInsertTemplate()` |
| `keymap.go` | `KeyMap` struct and `DefaultKeyMap()` |

### Message Routing in app.go


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sadopc/gottp](https://github.com/sadopc/gottp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
