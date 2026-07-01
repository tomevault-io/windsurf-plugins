---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: prox (Process Manager TUI)

A Terminal User Interface (TUI) application that replicates pm2 functionality - providing universal process management, monitoring, and control for applications in any language (Node.js, Python, Go, Rust, etc.).

## Current Status

**Phase 1 Complete:** Core process management, TUI dashboard, and real-time metrics collection are fully functional.

## Tech Stack

- **Language**: Go
- **TUI Framework**: Bubbletea + Bubbles + Lipgloss
- **Metrics**: gopsutil (github.com/shirou/gopsutil)
- **File Watching**: fsnotify

## Project Initialization

To start the project:

```bash
# Initialize Go module
go mod init github.com/yourusername/prox

# Install core dependencies
go get github.com/charmbracelet/bubbletea
go get github.com/charmbracelet/bubbles
go get github.com/charmbracelet/lipgloss
go get github.com/shirou/gopsutil/v3
go get github.com/fsnotify/fsnotify
go get github.com/spf13/cobra  # for CLI commands
```

## Planned Architecture

```
prox/
├── main.go                 # Entry point
├── cmd/                    # CLI commands (cobra)
│   ├── root.go
│   ├── start.go
│   ├── stop.go
│   ├── restart.go
│   └── tui.go
├── internal/
│   ├── config/            # Configuration management
│   ├── process/           # Process lifecycle & management
│   │   ├── manager.go     # Core process manager
│   │   ├── metrics.go     # Metrics collection
│   │   ├── restart.go     # Restart policies
│   │   └── watch.go       # File watching
│   ├── logs/              # Log management
│   │   ├── collector.go
│   │   └── rotation.go
│   ├── storage/           # Persistence (~/.prox/)
│   └── tui/               # TUI components (Bubbletea)
│       ├── app.go         # Main app model
│       ├── dashboard.go   # Process list view
│       ├── detail.go      # Process detail view
│       ├── logs.go        # Log viewer
│       ├── monitor.go     # Live monitor view
│       └── styles.go      # Lipgloss styles
└── pkg/                   # Public packages (if needed)
```

## Implemented Features

### ✅ Phase 1: Foundation (COMPLETE)
- Process spawning using `os/exec` with auto-detected interpreters
- Full process lifecycle (start, stop, restart, delete)
- PID tracking and status management
- State persistence in `~/.prox/state.json`
- Automatic reconnection to running processes on prox restart
- Graceful shutdown (SIGTERM → SIGKILL fallback)

### ✅ Phase 2: Monitoring (COMPLETE)
- Real-time metrics (CPU, memory, uptime) via gopsutil
- Process status tracking (online, stopped, errored, restarting)
- Beautiful TUI dashboard with live metrics updates (Bubbletea + Lipgloss)
- Restart counting
- Keyboard navigation and process controls

### 🚧 Phase 3: Logging (TODO)
1. Capture stdout/stderr from processes
2. Log viewer component in TUI
3. Log rotation (max size, max files)
4. Log search/filtering

### 🚧 Phase 4: Advanced (TODO)
1. Restart policies (always, on-failure, exponential backoff)
2. Crash loop prevention
3. File watching and auto-reload (fsnotify)
4. Configuration file support (JSON/YAML)
5. Process clustering (multiple instances)

## Key Design Patterns

### Process Management
- Use process groups for proper signal propagation
- Graceful shutdown: SIGTERM → wait → SIGKILL
- Track process state in memory + persist to disk
- Handle orphaned processes on manager restart

### TUI (Bubbletea)
- Follow Elm Architecture: Model → Update → View
- Use `tea.Cmd` for async operations (metrics collection, log streaming)
- Separate UI state from business logic
- Component-based views for reusability

### Metrics Collection
- Goroutine per process for concurrent monitoring
- Cache metrics with configurable refresh rate (default: 1s)
- Use channels for metric updates to TUI
- Handle processes that exit during metric collection

### Storage Strategy
```
~/.prox/
├── config.json          # Global settings
├── processes/           # Process configs (one JSON per process)
├── logs/               # Process logs (auto-rotate)
├── pids/               # PID files
└── state.json          # Current process states
```

## Build & Development Commands

```bash
# Build the binary
go build -o prox .

# Install locally
go install

# Run all tests
go test ./...

# Run with race detection
go test -race ./...

# Run specific package tests
go test ./internal/process/...

# Run with coverage
go test -cover ./...

# Tidy dependencies
go mod tidy
```

## Usage

```bash
# Launch interactive TUI (default)
./prox

# CLI mode - start a process
./prox start <script> [--name <name>] [--cwd <dir>] [-i <interpreter>]

# List all processes
./prox list

# Stop a process
./prox stop <name|id>

# Restart a process
./prox restart <name|id>
```

### TUI Keyboard Shortcuts

- `↑/k` - Move selection up
- `↓/j` - Move selection down
- `r` - Restart selected process
- `s` - Stop selected process
- `d` - Delete selected process
- `R` - Refresh process list
- `q` - Quit

## Important Implementation Notes

### Cross-platform Process Management
- Unix: Use `syscall.SysProcAttr` with `Setpgid: true` for process groups
- Windows: Different signal handling - use `taskkill` or WMI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craigderington/prox](https://github.com/craigderington/prox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
