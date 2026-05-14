---
trigger: always_on
description: This file provides guidance to AI assistants when working with the SimTool codebase. It consolidates project knowledge, development guidelines, and current status.
---

# CLAUDE.md

This file provides guidance to AI assistants when working with the SimTool codebase. It consolidates project knowledge, development guidelines, and current status.

## Project Overview

This is a Go project called `simtool` that uses the Bubble Tea framework to create a terminal UI application. The application displays a list of iOS simulators installed on the system and allows navigation with arrow keys or vim-style j/k keys.

## Project Structure

```
simtool/
├── cmd/simtool/        # Application entry point
│   └── main.go           # Main function
├── internal/
│   ├── config/         # Configuration and theme management
│   │   ├── config.go      # Config loading and theme selection
│   │   ├── config_test.go # Config tests
│   │   ├── detect.go      # Terminal theme detection initialization
│   │   ├── detect_test.go # Detection tests
│   │   ├── paths.go       # Config file paths
│   │   ├── styles.go      # Style generation from themes
│   │   ├── terminal_osc.go    # OSC escape sequence queries
│   │   ├── terminal_query.go  # Terminal background detection
│   │   ├── theme.go       # Theme color extraction and live detection
│   │   └── theme_test.go  # Theme tests
│   ├── simulator/      # Simulator types and fetching logic
│   │   ├── simulator.go   # Core types and interfaces
│   │   ├── simulator_test.go
│   │   ├── fetcher.go     # xcrun simctl integration
│   │   ├── fetcher_test.go
│   │   ├── app.go         # App information and listing
│   │   ├── app_test.go
│   │   ├── files.go       # File browsing and operations
│   │   ├── files_test.go
│   │   ├── files_darwin.go # macOS-specific file operations
│   │   ├── files_other.go  # Stub for other platforms
│   │   ├── viewer.go      # File content viewing
│   │   └── viewer_svg_test.go
│   ├── tui/           # Terminal UI components
│   │   ├── model.go       # Bubble Tea model with theme state
│   │   ├── model_test.go
│   │   ├── update.go      # Message handling and theme changes
│   │   ├── update_test.go # Update and theme change tests
│   │   ├── view.go        # Main view orchestrator
│   │   ├── view_test.go
│   │   ├── view_file.go   # File-specific view logic
│   │   ├── viewport.go    # Scrolling logic
│   │   ├── viewport_test.go
│   │   ├── keys.go        # Key bindings
│   │   └── components/    # Reusable UI components
│   │       ├── layout.go         # Base layout (title, content, footer)
│   │       ├── simulator_list.go # Simulator list view
│   │       ├── app_list.go       # App list view
│   │       ├── all_apps_list.go  # All apps combined view
│   │       ├── file_list.go      # File browser view
│   │       ├── database_table_list.go    # Database table list view
│   │       ├── database_table_content.go # Database table content view  
│   │       └── file_viewer/      # File viewer components
│   │           ├── viewer.go     # Main file viewer
│   │           ├── text.go       # Text file viewer
│   │           ├── image.go      # Image file viewer
│   │           ├── binary.go     # Binary file viewer
│   │           ├── archive.go    # Archive file viewer
│   │           └── database.go   # Database file viewer
│   └── ui/            # UI styles and formatting
│       ├── styles.go      # Dynamic style functions
│       ├── styles_test.go # Style function tests
│       ├── format.go      # Formatting helpers
│       └── format_test.go
├── scripts/
│   └── coverage-badge.sh  # Generate coverage badge
├── .gitignore
├── CLAUDE.md          # Project guidance for AI assistants
├── CLAUDE.local.md    # Private project instructions
├── go.mod
├── go.sum
├── Makefile           # Build automation
└── README.md          # User documentation
```

## Development Commands

### Building the Application
```bash
make build

# Or directly
go build -o simtool ./cmd/simtool
```

### Running the Application
```bash
make run

# Or directly
go run ./cmd/simtool

# Or after building
./simtool

# Start with all apps view (shows all apps from all simulators)
./simtool --apps
# or
./simtool -a
```

Note: This is a TUI application that requires a proper terminal environment. It won't run properly in environments without TTY support.

### Testing
```bash
make test
```

### Other Commands
```bash
# Format code
make fmt

# Clean build artifacts
make clean

# Install to $GOPATH/bin
make install

# Download dependencies
make deps

# Build for multiple platforms
make build-all
```

### Build System

The Makefile includes build-time version injection:
- Automatically uses git tags/commits for version information
- Injects commit hash, build date, and builder information
- Version format: `git describe --tags --always --dirty`
- Shows `-dirty` suffix if there are uncommitted changes

Example version output:
```
simtool version v1.0.0
  commit: abc123f
  built:  2025-06-07T12:00:00Z
  by:     username
```

## Architecture

The application follows clean architecture principles with clear separation of concerns:

### Packages

1. **internal/config**: Configuration management
   - Loads user configuration from `~/.config/simtool/config.toml`
   - Merges user settings with defaults

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [azizuysal/simtool](https://github.com/azizuysal/simtool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
