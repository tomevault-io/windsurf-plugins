---
trigger: always_on
description: TUIOS (Terminal UI Operating System) is a terminal-based window manager built in Go using the Charm stack (Bubble Tea v2, Lipgloss v2). It provides vim-like modal interface, workspace support, mouse interaction, and SSH server mode.
---

# AGENTS.md - Agent Guide for TUIOS

## Project Overview

TUIOS (Terminal UI Operating System) is a terminal-based window manager built in Go using the Charm stack (Bubble Tea v2, Lipgloss v2). It provides vim-like modal interface, workspace support, mouse interaction, and SSH server mode.

**Note:** The web terminal functionality is provided by the separate `tuios-web` binary for security isolation. See `cmd/tuios-web/` and [docs/WEB.md](docs/WEB.md) for details.

## Essential Commands

### Build & Run

```bash
# Build from source
go build -o tuios ./cmd/tuios
go build -o tuios-web ./cmd/tuios-web

# Run directly
go run ./cmd/tuios
go run ./cmd/tuios-web

# Run with debug logging
go run ./cmd/tuios --debug
go run ./cmd/tuios-web --debug

# Run tests
go test ./...

# Run specific package tests
go test ./internal/config/...
go test ./internal/tape/...

# Run with race detection
go test -race ./...
```

### Development with Nix

```bash
nix develop    # Enter development shell
nix build      # Build package
nix run        # Run directly
```

### Docker

```bash
docker build -t tuios .
docker run -it --rm tuios
```

## Code Organization

```
tuios/
├── cmd/tuios/              # CLI entry point (main.go with cobra commands)
├── cmd/tuios-web/          # Web terminal server binary (separate for security)
├── internal/
│   ├── app/                # Core window manager, OS model, rendering
│   │   ├── os.go           # Central state (OS struct), window lifecycle
│   │   ├── render.go       # View generation, layer composition
│   │   ├── update.go       # Bubble Tea Update() handler
│   │   ├── stylecache.go   # LRU style caching (40-60% allocation reduction)
│   │   ├── workspace.go    # Multi-workspace support (1-9)
│   │   └── animations.go   # Visual transitions
│   ├── config/             # Configuration and keybindings
│   │   ├── userconfig.go   # TOML config loading, defaults
│   │   ├── registry.go     # Keybind action lookup
│   │   └── validation.go   # Config validation
│   ├── input/              # Input handling and modal routing
│   │   ├── handler.go      # Main input coordinator
│   │   ├── keyboard.go     # Key event dispatch
│   │   ├── mouse.go        # Mouse interactions
│   │   ├── actions.go      # 40+ action handlers
│   │   └── copymode_*.go   # Vim-style copy mode (50+ motions)
│   ├── terminal/           # Terminal window management
│   │   ├── window.go       # Window struct, PTY lifecycle
│   │   └── pty_*.go        # Platform-specific PTY (unix/windows)
│   ├── vt/                 # Terminal emulation (ANSI/VT100)
│   │   ├── emulator.go     # Parser state machine
│   │   ├── screen.go       # Screen buffer management
│   │   ├── csi_*.go        # CSI sequence handlers
│   │   └── scrollback.go   # 10,000 line history
│   ├── tape/               # Tape scripting automation
│   │   ├── lexer.go        # Tokenizer
│   │   ├── parser.go       # AST generation
│   │   ├── executor.go     # Command execution
│   │   └── player.go       # Playback engine
│   ├── server/             # SSH server (Wish v2)
│   ├── theme/              # Color theming
│   ├── layout/             # Window tiling algorithms
│   ├── pool/               # Memory pooling
│   └── ui/                 # Animation system
├── docs/                   # Documentation
│   ├── ARCHITECTURE.md     # Technical architecture diagrams
│   ├── KEYBINDINGS.md      # Complete keybinding reference
│   ├── CONFIGURATION.md    # Config options
│   └── CLI_REFERENCE.md    # CLI flags and commands
├── examples/               # Tape script examples
└── nix/                    # Nix packaging
```

## Architecture Patterns

### Bubble Tea MVU Pattern

TUIOS follows Model-View-Update:
- **Model**: `app.OS` struct in `internal/app/os.go`
- **View**: `OS.View()` in `internal/app/render.go`
- **Update**: `OS.Update()` in `internal/app/update.go`

### Modal Input System

Two primary modes:
1. **WindowManagementMode**: Window manipulation, navigation
2. **TerminalMode**: Input forwarded to focused terminal PTY

Input routing: `internal/input/handler.go` → mode-specific handlers

### Prefix Key System (tmux-style)

Leader key (`Ctrl+B` by default) activates prefix mode with sub-menus:
- `Ctrl+B` then `w` → Workspace prefix
- `Ctrl+B` then `m` → Minimize prefix
- `Ctrl+B` then `t` → Tiling prefix
- `Ctrl+B` then `D` → Debug prefix
- `Ctrl+B` then `T` → Tape manager prefix

### Window Lifecycle

1. `OS.AddWindow()` creates window with PTY
2. PTY spawns shell process with I/O polling goroutines
3. VT emulator parses ANSI output
4. Screen buffer updates trigger render
5. `OS.DeleteWindow()` cleans up PTY and removes window

## Key Dependencies

- **Bubble Tea v2** (`charm.land/bubbletea/v2`) - TUI framework
- **Lipgloss v2** (`charm.land/lipgloss/v2`) - Styling
- **Wish v2** (`charm.land/wish/v2`) - SSH server
- **Ultraviolet** (`github.com/charmbracelet/ultraviolet`) - Terminal emulation base
- **Cobra** (`github.com/spf13/cobra`) - CLI commands
- **xpty** (`github.com/charmbracelet/x/xpty`) - Cross-platform PTY


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gaurav-Gosain/tuios](https://github.com/Gaurav-Gosain/tuios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
