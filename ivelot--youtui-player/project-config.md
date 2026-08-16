---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, Gemini, etc.) when working with code in this repository. `CLAUDE.md` and `GEMINI.md` are symlinks to this file.
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Gemini, etc.) when working with code in this repository. `CLAUDE.md` and `GEMINI.md` are symlinks to this file.

## Stack

- **Go 1.24** (toolchain `go1.24.7`, per `go.mod`)
- Deps (from `go.mod`/`go.sum`): `github.com/rivo/tview`, `github.com/gdamore/tcell/v2` v2.7.4, `github.com/BurntSushi/toml` v1.5.0, `github.com/nfnt/resize`
- No database, no queue, no external API service — the app shells out to local CLI tools instead (see Runtime Dependencies)
- Packaged for Arch Linux via `PKGBUILD` (AUR)

## Commands

```bash
make build       # Compile the project
make run         # Build and run
make test        # Run all tests (go test ./...)
make fmt         # Format code (go fmt ./...)
make vet         # Static analysis (go vet ./...)
make check-deps  # Verify runtime deps (mpv, yt-dlp, socat, ffmpeg)
make clean       # Remove compiled binaries
make deps        # Download and tidy Go modules
```

Run a single test: `go test ./internal/... -run TestName`

**Note:** `go`/`make` toolchain execution was permission-gated in the adoption sandbox and could not be interactively approved, so `make build`/`make test`/`make vet` were not actually run during this adoption pass. Treat them as documented-but-unverified until run once in a normal shell.

## Runtime Dependencies

The app shells out to external tools at runtime:
- `mpv` — media player backend
- `yt-dlp` — YouTube search and extraction
- `socat` — IPC communication with mpv via Unix socket
- `ffmpeg` — used by `yt-dlp` to extract audio to MP3 and merge video into MP4 on download

## Environment variables

No secrets or required env vars. The code reads a few optional XDG base-dir overrides, all with hardcoded fallbacks:

| Variable | Used in | Fallback |
|---|---|---|
| `XDG_CONFIG_HOME` | `internal/config/config.go` | `~/.config` |
| `XDG_STATE_HOME` | `internal/config/state.go` | `~/.local/state` |
| `XDG_CACHE_HOME` | `internal/ui/thumbnail.go` | `~/.cache` |
| `XDG_DATA_HOME` | `internal/ui/download.go` | `~/.local/share` |
| `XDG_DOWNLOAD_DIR` | `internal/ui/download.go` | `~/Downloads` |
| `LC_ALL` / `LC_MESSAGES` / `LANG` | `internal/config/config.go` | English (`en`) |

See `.env.example` for a reference copy of these (all optional).

## Architecture

**Module:** `github.com/IvelOt/youtui-player`

Three internal packages:

- `internal/ui/` — Main TUI layer. The central `SimpleApp` struct (defined in `app.go`) owns all state and UI components. All subsystems are wired in `setup.go`; input handling lives in `handlers.go`.
- `internal/search/` — YouTube search via `yt-dlp` subprocess; parses JSON output into `Result` structs. Three entry points: `SearchVideos()`, `GetPlaylistVideos()`, `GetVideoDetails()`.
- `internal/config/` — TOML config (`~/.config/youtui-player/youtui.conf`) and JSON session state (`~/.local/state/youtui-player/state.json`).

**Key data flow:**

1. User search → `search.SearchVideos()` spawns `yt-dlp`, returns `[]Result`
2. Playback → `playTrackSimple()` spawns `mpv` with `--input-ipc-server`; progress polling uses `socat` to query the IPC socket
3. State is auto-saved to JSON after changes and restored on startup. `PlayerState` persists search term, search results, playlist, scroll positions, current page, and playlist/play modes.

**Concurrency:** `SimpleApp.mu` (sync.Mutex) guards shared state. All UI mutations must go through `app.QueueUpdateDraw()`; background goroutines (playback, thumbnail fetching, search) must never touch UI primitives directly.

**yt-dlp health check:** `health_check.go` checks the installed `yt-dlp` version at startup and warns if it is more than 14 days old.

**Theme system:** Four built-in Catppuccin variants (Latte, Frappé, Macchiato, Mocha), a `Terminal` theme that inherits the host palette (`tcell.ColorDefault` backgrounds/text + ANSI accents), plus custom themes loaded from TOML files. `colorTag()` emits `-` (reset) for `ColorDefault` so the Terminal theme renders natively in dynamic-color markup. Theme and language changes apply in real time via `applyTheme()`/`applyLanguage()`; because `HelpView` bakes theme colors into its text at construction, `applyTheme()` rebuilds it (`setupHelpView()`) so the help screen follows the new palette. See `THEMES.md` for the color palette format.

**i18n:** `internal/ui/i18n.go` provides PT-BR and EN string packs, auto-detected from `LC_ALL`/`LANG` environment variables or the config file.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IvelOt/youtui-player](https://github.com/IvelOt/youtui-player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
