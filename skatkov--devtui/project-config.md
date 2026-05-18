---
trigger: always_on
description: This file helps AI agents (and developers) understand how to work effectively in the DevTUI codebase.
---

# AGENTS.md - DevTUI Development Guide

This file helps AI agents (and developers) understand how to work effectively in the DevTUI codebase.

## Project Overview

**DevTUI** is a Swiss-army knife terminal app for developers, providing both CLI and TUI (Terminal User Interface) modes for common developer utilities like JSON formatting, base64 encoding/decoding, UUID generation, and more.

- **Language**: Go 1.25.3
- **CLI Framework**: Cobra (github.com/spf13/cobra)
- **TUI Framework**: Bubble Tea (github.com/charmbracelet/bubbletea)
- **Build Tool**: goreleaser
- **Testing**: Standard Go testing (`testing` package)
- **Linting**: golangci-lint with formatters gofumpt and goimports

## Essential Commands

### Build & Run
```bash
# Build the project
go build -v ./...

# Run the TUI (main mode)
go run main.go

# Run a specific CLI command
go run main.go jsonfmt < testdata/example.json
go run main.go base64 "hello world"
go run main.go --help
```

### Testing
```bash
# Run all tests with race detection (this is what CI uses)
go test -v -failfast -race ./...

# Run specific package tests
go test -v ./cmd/...
go test -v ./internal/ui/...

# Run specific test
go test -v -run TestJson2toonCmd ./cmd/
```

### Linting & Formatting
```bash
# Run golangci-lint (what CI uses)
golangci-lint run

# The linters will auto-format with gofumpt and goimports
# Enabled linters: bodyclose, goconst, gomoddirectives, goprintffuncname, 
# misspell, nakedret, nilerr, noctx, nolintlint, prealloc, rowserrcheck, 
# sqlclosecheck, tparallel, unconvert, unparam, whitespace, perfsprint
```

### Go Module Management
```bash
# Tidy dependencies (goreleaser runs this before builds)
go mod tidy

# Generate code (goreleaser runs this before builds)
go generate ./...
```

### Documentation Generation
```bash
# Generate both CLI and TUI documentation (for website)
cd docs && go run *.go

# Generate only CLI docs
cd docs && go run cli-docs.go docs.go

# Generate only TUI docs  
cd docs && go run tui-docs.go docs.go
```

### Release
```bash
# Build release with goreleaser (creates binaries for linux/darwin/windows)
goreleaser release --snapshot --clean

# The project auto-releases via GitHub Actions on tags
```

## Project Structure

```
devtui/
├── main.go                    # Entry point - just calls cmd.Execute()
├── cmd/                       # CLI command implementations (Cobra)
│   ├── root.go               # Root command, launches TUI by default
│   ├── base64.go             # CLI commands like base64, jsonfmt, etc.
│   ├── jsonfmt.go
│   ├── json2toon.go
│   ├── jsonrepair.go
│   ├── *_test.go             # CLI command tests
│   └── ...                   # ~12 CLI commands total
├── tui/                      # TUI module implementations (Bubble Tea)
│   ├── root/                 # Root TUI screen with menu
│   │   ├── root.go          # Root model managing screen switching
│   │   └── list.go          # Menu list with usage tracking
│   ├── json/                # Each TUI module in its own package
│   ├── yaml/
│   ├── base64-encoder/
│   ├── json2toon/
│   └── ...                  # ~27 TUI modules total
├── internal/                # Shared internal packages
│   ├── ui/                  # TUI UI components & common types
│   │   ├── ui.go           # CommonModel, ReturnToListMsg, constants
│   │   ├── base_pager_model.go  # BasePagerModel (base for most TUI screens)
│   │   └── style.go        # Lipgloss styles
│   ├── editor/             # External editor integration
│   ├── base64/             # Base64 encode/decode logic
│   ├── csv2md/             # CSV to markdown conversion
│   └── textanalyzer/       # Text analysis utilities
├── docs/                   # Documentation generators
├── testdata/               # Test fixtures (JSON, CSV, TOML, etc.)
├── site/                   # Jekyll website (generated docs)
├── .golangci.yml          # Linter config
├── .goreleaser.yaml       # Release config
└── .github/workflows/     # CI/CD
    ├── build.yml          # Build & test on push/PR
    ├── lint.yml           # Linting with golangci-lint
    └── jekyll-gh-pages.yml # Deploy docs to GitHub Pages
```

## Code Architecture

### Dual Interface Pattern

DevTUI provides **both CLI and TUI** for most utilities:

1. **CLI Commands** (`cmd/`): For piping/scripting
   - Use Cobra framework
   - Read from stdin or args
   - Write to stdout
   - Example: `devtui jsonfmt < file.json > output.json`

2. **TUI Modules** (`tui/`): For interactive use
   - Use Bubble Tea framework
   - Full-screen interactive UIs
   - Example: Run `devtui` (no args) to see menu

**Not all features have both interfaces yet** - some are CLI-only or TUI-only.

### TUI Architecture

**Root Screen** (`tui/root/`):
- `root.go`: Manages current view and screen switching
- `list.go`: Main menu with fuzzy search and usage tracking
- Menu items are sorted by usage frequency (stored in `~/.config/devtui/usage_stats.json`)

**TUI Modules** (`tui/*/`):
- Each tool is a separate package (e.g., `tui/json`, `tui/yaml`)
- Most inherit from `BasePagerModel` (`internal/ui/base_pager_model.go`)
- Common pattern: Title constant, Model struct, Init/Update/View methods

**BasePagerModel** provides:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skatkov/devtui](https://github.com/skatkov/devtui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
