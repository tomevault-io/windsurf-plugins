---
trigger: always_on
description: This file provides guidance to AI coding assistants.
---

# AGENTS.md

This file provides guidance to AI coding assistants.

## AI Guidance

* After receiving tool results, carefully reflect on their quality and determine optimal next steps before proceeding. Use your thinking to plan and iterate based on this new information, and then take the best next action.
* For maximum efficiency, whenever you need to perform multiple independent operations, invoke all relevant tools simultaneously rather than sequentially.
* Before you finish, please verify your solution
* Do what has been asked; nothing more, nothing less.
* NEVER create files unless they're absolutely necessary for achieving your goal.
* ALWAYS prefer editing an existing file to creating a new one.
* NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.
* When you update or modify core context files, also update markdown documentation and memory bank
* When asked to commit changes, exclude AGENTS.md and AGENTS-*.md referenced memory bank system files from any commits.

## Memory Bank System

This project uses a structured memory bank system with specialized context files. Always check these files for relevant information before starting work:

### Core Context Files

* **AGENTS-activeContext.md** - Current session state, goals, and progress (if exists)
* **AGENTS-patterns.md** - Established code patterns and conventions (if exists)
* **AGENTS-decisions.md** - Architecture decisions and rationale (if exists)
* **AGENTS-troubleshooting.md** - Common issues and proven solutions (if exists)
* **AGENTS-config-variables.md** - Configuration variables reference (if exists)
* **AGENTS-temp.md** - Temporary scratch pad (only read when referenced)

**Important:** Always reference the active context file first to understand what's currently being worked on and maintain session continuity.

### Memory Bank System Backups

When asked to backup Memory Bank System files, you will copy the core context files above and @.agents settings directory to directory @/path/to/backup-directory. If files already exist in the backup directory, you will overwrite them.

## Project Overview

DankMaterialShell is a complete desktop environment for Wayland compositors, built as a **monorepo** with two main components:

**1. Go Backend (core/)** - System integration, IPC server, and CLI tools (~118,000 lines)
**2. QML Frontend (quickshell/)** - UI layer consuming the backend's IPC API

**Architecture**: The Go backend provides all system integration via IPC (Inter-Process Communication), while QML services act as thin wrappers that communicate with the backend. This separation allows for robust system integration while maintaining a reactive, modern UI.

**Compositor Support**: Niri, Hyprland, MangoWC, Sway, labwc, Scroll (6 compositors supported)
**Distribution Support**: Arch, Fedora, Debian, Ubuntu, openSUSE, Gentoo (6 distributions supported)

## Technology Stack

### Backend (core/)
- **Go 1.24+** - System integration and backend services
- **Wayland Protocols** - Display management, screenshots, clipboard, workspaces
- **D-Bus** - Bluetooth, NetworkManager, systemd-logind, desktop portals
- **IPC Server** - Unix socket JSON API for QML ↔ Go communication
- **CLI Tools** - `dms` command with 20+ subcommands, `dankinstall` TUI installer

### Frontend (quickshell/)
- **QML (Qt Modeling Language)** - UI components and visual presentation
- **Quickshell Framework** - QML-based desktop shell framework
- **Qt/QtQuick** - UI rendering and controls
- **Matugen** - Dynamic theming system for wallpaper-based colors

## Development Commands

### Backend (Go)

```bash
cd core/

# Build
make                 # Build dms CLI (bin/dms)
make dankinstall     # Build installer (bin/dankinstall)
make test            # Run tests
make dist            # Build distribution binaries (no update/greeter features)

# Install
sudo make install    # Install to /usr/local/bin/dms

# Development
gofmt -w .           # Format Go code
go mod tidy          # Clean up dependencies
golangci-lint run    # Run linter

# Run dms CLI
./bin/dms run        # Start shell via dms daemon
./bin/dms ipc <cmd>  # Send IPC command to running shell
./bin/dms --help     # View all commands
```

### Frontend (QML)

```bash
cd quickshell/

# Run the shell (requires dms backend running or use 'dms run')
quickshell -p shell.qml
qs -p .              # Shorthand
qs -v -p shell.qml   # Verbose debugging

# Code formatting and linting
qmlfmt -t 4 -i 4 -b 250 -w /path/to/file.qml  # Format QML (don't use qmlformat)
make -C .. lint-qml  # From quickshell/, call the repo-root lint target; requires the generated .qmlls.ini VFS from `qs -p .`
./qmlformat-all.sh   # Format all QML files
```

## Architecture Overview

### Monorepo Structure

The project is organized as a monorepo with clear separation between backend and frontend:

```
DankMaterialShell/
├── core/               # Go backend (~118,000 lines)
│   ├── cmd/            # Binary entrypoints
│   │   ├── dms/        # Main CLI with 20+ commands
│   │   └── dankinstall/# TUI installer
│   ├── internal/       # System integration packages (23 packages)
│   │   ├── clipboard/  # Clipboard history (ext-data-control-v1)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AvengeMedia/DankMaterialShell](https://github.com/AvengeMedia/DankMaterialShell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
