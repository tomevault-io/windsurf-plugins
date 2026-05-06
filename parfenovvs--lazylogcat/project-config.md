---
trigger: always_on
description: TUI application for viewing Android logcat logs, built with Go 1.25+ and Bubble Tea v2. Requires `adb` in PATH.
---

# Agent Guidelines for lazylogcat

TUI application for viewing Android logcat logs, built with Go 1.25+ and Bubble Tea v2. Requires `adb` in PATH.

## Build, Test & Run

```bash
# Build
go build -v ./...              # Build all packages
go build -o lazylogcat .       # Build binary

# Run
go run main.go                       # Launch TUI
go run main.go --debug               # Debug logging to .lazylogcat.log

# Test
go test -v ./...                                           # All tests
go test -v -race ./...                                     # With race detector (CI default)
go test -v -coverprofile=coverage.out ./...                # With coverage
go test -v ./internal/config                               # Single package
go test -v -run TestDefaultConfig ./internal/config        # Single test
go test -v -run TestAppend/ExceedCapacity ./internal/util  # Single subtest
go tool cover -func=coverage.out                           # Coverage report

# Lint & Verify
go fmt ./...       # Format
go vet ./...       # Static analysis
go mod tidy        # Clean go.mod
go mod verify      # Verify deps
```

### Frontend (web-ui) — requires `bun`

```bash
(cd web-ui && bun install)        # Install JS dependencies (one-time setup)
go generate ./internal/web/...    # Build frontend into internal/web/static/
bun run --cwd web-ui dev          # Dev server with HMR (proxies /api, /ws to :8321)
```

## Architecture

**TUI entry flow**: `main.go` -> `cmd/root.go` (Cobra CLI) -> `app/app.go` (PreLaunchChecks, LaunchTUI) -> `mainui/tui.go`

**Web entry flow**: `main.go` -> `cmd/web.go` (Cobra CLI) -> `internal/web/server.go` (HTTP + WebSocket server, log streaming, embedded static UI)

Uses the **Elm Architecture** (Model-View-Update) via Bubble Tea. `MainModel` is the root state machine coordinating views via `sessionState` iota enum. Custom messages (`tea.Msg`) drive all state transitions; `tea.Cmd` handles async operations.

## Project Structure

```
├── cmd/                 # CLI commands (cobra): root.go, version.go, web.go, skill.go, …
├── skills/              # Bundled agent skill(s), embedded into the binary (`//go:embed`)
│   └── lazylogcat/      # SKILL.md for Cursor / Claude Code (see `lazylogcat skill install`)
├── internal/
│   ├── app/            # App lifecycle (pre-launch checks, debug logging setup)
│   ├── config/         # Configuration (layered: defaults -> global -> project -> local)
│   ├── model/          # Data models (Device, Filter, LogLine, Command, CommandData, CommandGroup, OutputPrefs, Columns, Size, Level, TextFilter, TextFilterMode)
│   ├── tui/            # Shared UI types, messages (msg.go), overlay utils, toast
│   │   ├── commandui/ # Command palette: dialogs, single/multi select, text input
│   │   ├── logcatui/  # Log streaming view with visual mode and search
│   │   ├── mainui/    # Root model, state machine coordinating views
│   │   └── theme/     # ANSI-16 color palette and style constructors
│   ├── util/           # ADB wrapper, logcat parser, logcat reader, ring buffer, clipboard, editor, config-to-model mappers
│   └── web/            # HTTP + WebSocket server (experimental web UI backend)
├── web-ui/              # React frontend source (built output embedded into binary)
├── config.schema.json   # JSON Schema for config files
└── main.go              # Entry point -> cmd.Execute()
```

## Web UI (experimental)

> **Warning:** The web experience is experimental and may change or break without notice.

### Command

```
lazylogcat web [flags]
  --port int     Port to listen on (default 8321)
  --demo         Run with a fake device and synthetic log lines (no adb required)
  --pkg string   Filter by package name (contains match, overrides config)
  --tag string   Filter by log tag (contains match, overrides config)
  --text string  Filter by log text (contains match, overrides config)
  --debug        Enable debug logging (inherited from root)
```

Starts an HTTP server, opens the browser automatically, and waits for SIGINT/SIGTERM to shut down.

### HTTP Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /api/devices` | Returns connected ADB devices as JSON |
| `GET /api/config` | Returns resolved app config as JSON |
| `GET /ws` | WebSocket for real-time log streaming |
| `GET /` | Serves embedded static frontend |

### WebSocket Protocol

Client→server message types: `connect`, `disconnect`, `updateFilter`, `listDevices`

Server→client message types: `lines`, `connected`, `disconnected`, `devices`, `error`, `clearLines` (sent after `updateFilter` so the client can drop buffered lines)

Each WebSocket connection gets its own `Session` with a `LogcatReader` and a 10,000-line `RingBuffer`. Log lines are drained every 50ms and sent in batches.

### Frontend Stack

- **Framework:** React v19 + TypeScript
- **Build tool:** Vite v6 via `bun`
- **Styling:** Emotion (`@emotion/react`, `@emotion/styled`)
- **Output:** `internal/web/static/` — embedded into the Go binary via `//go:embed static/*`
- **UI primitives:** MUI v7 (`@mui/material`, `@mui/icons-material`)
- **Virtualization:** `@tanstack/react-virtual` v3 — windowed log list

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parfenovvs/lazylogcat](https://github.com/parfenovvs/lazylogcat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
