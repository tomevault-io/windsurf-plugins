---
trigger: always_on
description: **ALWAYS create a new branch for any changes** - features, fixes, refactors, documentation, etc.
---

# Claude Code Assistant Guide for goplaying

## Development Workflow

### Branch Strategy
**ALWAYS create a new branch for any changes** - features, fixes, refactors, documentation, etc.

Branch naming conventions:
- **Features**: `feat/description` (e.g., `feat/volume-control`)
- **Bug fixes**: `fix/description` (e.g., `fix/redundant-image-decoding`)
- **Refactoring**: `refactor/description` (e.g., `refactor/split-main`)
- **Documentation**: `docs/description` (e.g., `docs/update-readme`)
- **Chores**: `chore/description` (e.g., `chore/update-deps`)

**Process**:
1. Create branch: `git checkout -b <type>/short-description`
2. Make changes and commit
3. Push and create PR when ready
4. Merge to main after review/approval

Never commit directly to main unless explicitly requested.

## Project Overview

**goplaying** is a cross-platform TUI (Terminal User Interface) music player status display written in Go. It shows currently playing music with album artwork, playback controls, and auto-extracted color themes.

- **Platforms**: macOS (MediaRemote + AppleScript), Linux (playerctl/MPRIS)
- **Frameworks**: Bubble Tea (TUI), Lipgloss (styling)
- **Special Features**: Kitty graphics protocol for album artwork, K-means color extraction, live config reload

## Key Files

### Core Application (Modular Architecture)
The application is split into focused modules for better maintainability:

- **main.go** (39 lines): Application entry point
  - Command-line flag definitions
  - Initializes configuration
  - Creates and runs Bubble Tea program
  
- **config.go** (146 lines): Configuration management
  - `Config` struct with Viper configuration management
  - `SafeConfig`: Thread-safe config wrapper with `sync.RWMutex`
  - `initConfig()`: Loads config from `~/.config/goplaying/config.yaml`
  - `watchConfigCmd()`: Live config reload via fsnotify
  - Global `config` variable (use `config.Get()` and `config.Set()` for thread safety)

- **model.go** (335 lines): Bubble Tea model and business logic
  - `SongData` struct: Current track metadata
  - `model` struct: Application state (song data, artwork, scrolling state)
  - `Init()`, `Update()`: Bubble Tea lifecycle methods
  - `fetchSongData()`: Background data fetching from media controller
  - `getCurrentPosition()`: Smooth position interpolation for progress bar
  - Message types: `tickMsg`, `fetchMsg`, `songDataMsg`

- **view.go** (168 lines): UI rendering
  - `View()`: Renders TUI with lipgloss styling
  - Handles artwork display with Kitty graphics protocol
  - Progress bar, scrolling text, help text
  - Dynamic status icons (play/pause/stop)

- **artwork.go** (248 lines): Image processing and color extraction
  - `extractDominantColor()`: K-means color extraction from artwork
  - `encodeArtworkForKitty()`: Converts artwork to Kitty graphics protocol
  - `supportsKittyGraphics()`: Terminal capability detection
  - Handles base64 encoding/decoding, image resizing, chunking

- **text.go** (32 lines): Text utilities
  - `formatTime()`: Converts seconds to MM:SS format
  - `scrollText()`: Unicode-aware text scrolling with looping

### Platform-Specific Media Controllers
- **media_darwin.go**: macOS implementation
  - `HybridController`: MediaRemote framework (via Swift helper) + AppleScript fallback
  - Artwork via temp file (AppleScript limitation)
  - Supports: Music, Spotify, any Now Playing source
  
- **media_linux.go**: Linux implementation
  - Uses `playerctl` command-line tool
  - MPRIS D-Bus protocol support
  - Artwork via `mpris:artUrl` metadata

- **media.go**: Interface definition
  - `MediaController` interface for cross-platform abstraction

### macOS Swift Helper
- **helpers/nowplaying/main.swift**: MediaRemote private framework wrapper
  - Returns base64-encoded artwork data
  - Required for broader app support beyond Music/Spotify
  - Build with: `cd helpers/nowplaying && make`
  - Requires: Xcode Command Line Tools (`xcode-select --install`)

### Configuration
- **config.example.yaml**: Configuration template
  - `ui.color`: Manual color (ANSI or hex)
  - `ui.color_mode`: "manual" or "auto" (extract from artwork)
  - `ui.max_width`: Border width
  - `artwork.enabled`: Toggle artwork display
  - `artwork.padding`: Space for artwork (columns)
  - `text.max_length_with_art` / `text.max_length_no_art`: Scrolling text width
  - `timing.ui_refresh_ms`: UI tick rate (100ms default)
  - `timing.data_fetch_ms`: Metadata fetch rate (1000ms default)
  
- Config location: `~/.config/goplaying/config.yaml`
- Live reload: Changes apply immediately via fsnotify

### Build System
- **Makefile**: Build targets
  - `make` or `make goplaying`: Build main binary
  - `make darwin`: Build main + Swift helper (macOS)
  - `make linux`: Build main binary only
  - `make fmt`: Run gofmt + goimports
  - `make lint`: Run golangci-lint
  - `make test`: Run tests
  
- **.golangci.yml**: Linter configuration
  - Enabled: errcheck, gofmt, goimports, govet, staticcheck, unused, gosimple

## Architecture Patterns

### Data Flow
1. **Timer loops**: Two concurrent tick loops (UI refresh 100ms, data fetch 1000ms)
2. **Background fetching**: `fetchSongData()` returns `tea.Cmd` for non-blocking I/O

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [justinmdickey/goplaying](https://github.com/justinmdickey/goplaying) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
