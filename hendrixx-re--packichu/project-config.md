---
trigger: always_on
description: > **Packichu** is a terminal-based (TUI) package management and AI-powered system inspection dashboard for Arch Linux.
---

# Packichu — Project Reference for AI Agents

> **Packichu** is a terminal-based (TUI) package management and AI-powered system inspection dashboard for Arch Linux.
> It lets users browse, inspect, AI-analyze, search, install, update, and batch-uninstall packages across multiple
> package managers (Pacman, AUR, Flatpak) in a unified, reactive interface.

---

## Quick Facts

| Key | Value |
|---|---|
| Language | Go 1.26 |
| Module | `packichu` |
| TUI Framework | [Bubble Tea](https://github.com/charmbracelet/bubbletea) + [Lip Gloss](https://github.com/charmbracelet/lipgloss) + [Bubbles](https://github.com/charmbracelet/bubbles) |
| AI Providers | **Google Gemini** (`gemini-2.5-flash`), **Groq** (`openai/gpt-oss-120b`), **OpenAI** (`gpt-4o-mini`), **Anthropic** (`claude-3-5-haiku-latest`) |
| Provider Detection | Automatic based on active API key in config (`GEMINI_API_KEY`, `GROQ_API_KEY`, etc.) |
| Config | `~/.config/packichu/config.env` (or `~/.config/packichu/.env`, `./.env`, `~/.packichu.env`) |
| Cache | `~/.cache/packichu/analysis.json` (keyed permanently by `pkg.Name`) |
| Binary | `packichu` (built with `go build`) |
| Target OS | Linux (Arch-based, uses `pacman`) |
| Optional Deps | `yay` or `paru` (for AUR search, install & upgrade support), `flatpak` |
| Color Themes | **Gruvbox Retro** (Default), **Catppuccin** (Mocha), **Monokai** (cycle with `t`) |

---

## Directory Structure

```
packichu/
├── main.go                  # Entry point — loads XDG config, starts Bubble Tea program
├── go.mod / go.sum          # Module definition and dependency lock
├── .env.example             # Template config file with Gemini, Groq, OpenAI, Anthropic examples
├── .gitignore
├── Makefile                 # Build, test, clean, install targets
├── aur/
│   └── PKGBUILD             # Arch Linux User Repository (AUR) package build definition
├── GEMINI.md                # AI Agent Reference & Architecture Guide (this file)
├── README.md                # User-facing documentation
│
└── internal/
    ├── ai/
    │   ├── analyzer.go      # Multi-provider AI integration (auto-detection, singleflight, pacing, backoff)
    │   └── analyzer_test.go # Deduplication, delay floor & provider auto-detection tests
    │
    ├── cache/
    │   ├── cache.go         # Thread-safe JSON file cache (RWMutex, package-name keying, auto-migration)
    │   └── cache_test.go    # Cache lookup, version persistence & migration tests
    │
    ├── pm/                  # Package Manager abstraction layer
    │   ├── package.go       # Package struct, Manager interface
    │   ├── detect.go        # Dynamic host package manager detection (Pacman, AUR, Flatpak)
    │   ├── detect_test.go   # Manager detection unit tests
    │   ├── pacman.go        # Pacman implementation (pacman -Qin parser, search, update, orphans)
    │   ├── aur.go           # AUR implementation (pacman -Qim, yay/paru helper search & install)
    │   ├── flatpak.go       # Flatpak implementation (list, search, install, update, cleanup)
    │   ├── fuzzy.go         # Fuzzy search and repository relevance ranking engine
    │   └── fuzzy_test.go    # Fuzzy scoring unit tests
    │
    └── tui/                 # Terminal UI (Bubble Tea)
        ├── model.go         # Model struct, Init(), tea commands, theme helpers
        ├── msgs.go          # Tea message types
        ├── update.go        # Update() — key handling, cross-manager sync worker, modal flows, upgrades
        ├── view.go          # View() — rendering sidebar, list, detail, highlighted command badge, modals
        ├── styles.go        # Theme definitions (Gruvbox Retro, Catppuccin, Monokai) & ApplyTheme()
        ├── theme_test.go    # Theme cycling & palette tests
        └── verdict_test.go  # Verdict extraction, command badge cleaning & markdown parsing tests
```

---

## Architecture

### 1. Entry Point & Config Loading (`main.go`)

- `loadConfig()` searches and loads configuration in standard priority order:
  1. `~/.config/packichu/config.env` (or `~/.config/packichu/.env`, `~/.config/packichu/config`)
  2. `~/.packichu.env`
  3. `./.env` (Current working directory)
  4. Executable directory `.env`
- Parses `KEY=VALUE` pairs, strips comments/quotes, and sets environment variables not already exported.
- Launches the Bubble Tea program with full alt-screen and mouse support.

---

### 2. Package Manager Layer (`internal/pm/`)

#### `Manager` Interface (`package.go`)

```go
type Manager interface {
    Name() string
    ListAll() ([]Package, error)
    GetPackage(name string) (*Package, error)
    UninstallCmd(names []string) []string
    UninstallOrphansCmd() []string
    GetOrphans() ([]string, error)
    InstallCmd(name string) []string
    UpdateCmd() []string
    UpdatePackagesCmd(names []string) []string
    Search(query string) ([]Package, error)
    RequiresSudo() bool
}
```

#### `Package` Struct

```go
type Package struct {
    Name          string
    Version       string
    Description   string
    Architecture  string
    Size          int64         // bytes
    InstallDate   time.Time
    BuildDate     time.Time
    InstallReason string        // "Explicitly installed" or "Installed as a dependency"
    Dependencies  []string

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hendrixx-RE/packichu](https://github.com/Hendrixx-RE/packichu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
