---
trigger: always_on
description: > A retro terminal music player (Go + Bubbletea). This file tells AI agents where things live, what conventions the codebase uses, and which skills to lean on.
---

# CLAUDE.md — cliamp

> A retro terminal music player (Go + Bubbletea). This file tells AI agents where things live, what conventions the codebase uses, and which skills to lean on.

## Extended context

**More narrative detail, design notes, and roadmap context for cliamp lives in `~/Documents/bjarne/projects/cliamp/`.** Read files in that directory when you need background that isn't captured in code or in `docs/`. Treat it as the project's long-form knowledge base (goals, decisions, TODOs). If a question feels strategic rather than tactical, check there first.

---

## What cliamp is

A TUI music player inspired by Winamp. Plays local files, HTTP streams, podcasts, and content from many providers: YouTube / YouTube Music, SoundCloud, Bilibili, Spotify, Xiaoyuzhou, Navidrome, Plex, Jellyfin, and a curated radio directory. Ships with a spectrum visualizer, 10-band parametric EQ, Lua plugin system, IPC remote control, and MPRIS/MediaRemote integration.

- Site: https://cliamp.stream
- Install: `curl -fsSL https://raw.githubusercontent.com/bjarneo/cliamp/HEAD/install.sh | sh`
- Entry point: `main.go` → `run(...)` wires providers, player, playlist, Lua plugin manager, IPC server, and the Bubbletea program.
- CLI built with `urfave/cli/v3` in `commands.go`.

---

## Architecture map

Top-level layout (each subdirectory below is a Go package):

| Path | Responsibility |
|------|----------------|
| `main.go`, `commands.go` | Entry point, CLI definition, subcommands (IPC clients: play/pause/next/seek/…) |
| `config/` | TOML config load/save, CLI overrides, provider-specific config blocks |
| `player/` | Audio engine. Decoding (FFmpeg, yt-dlp), DSP pipeline, 10-band EQ, ICY, gapless, platform audio devices (`audio_device_*.go`) |
| `playlist/` | Playlist model, shuffle/repeat, M3U/PLS encoding, tag reading, queue navigation |
| `provider/` | `interfaces.go` + `types.go`: the `Provider` contract every source implements |
| `external/<name>/` | One Go package per provider: `local`, `radio`, `navidrome`, `plex`, `jellyfin`, `spotify`, `ytmusic` |
| `resolve/` | Expand user arguments (files, dirs, M3U/PLS, URLs, search queries) into playable tracks |
| `ui/` | Bubbletea view layer: visualizers (`vis_*.go`), global styles, tick loop |
| `ui/model/` | The Bubbletea `Model`: state, update, keymap, overlays, file browser, search, EQ, seek, notifications, providers. This is the biggest directory — always start here for UI behavior |
| `luaplugin/` | Gopher-Lua VM wrapper + sandbox + plugin APIs (`api_player.go`, `api_fs.go`, `api_http.go`, `api_message.go`, `api_crypto.go`, …). Plugin visualizers live here too |
| `plugins/` | First-party bundled Lua plugins (`now-playing.lua`, `auto-eq.lua`, …) |
| `pluginmgr/` | `cliamp plugins install/remove/list` CLI: resolves GitHub/GitLab/Codeberg sources and a direct URL |
| `ipc/` | Unix-socket IPC: `server.go` listens; `client.go` + `protocol.go` drive subcommands like `cliamp pause` from outside the TUI |
| `mediactl/` | MPRIS (Linux, dbus) + NowPlaying (macOS) integration. `service_linux.go`, `service_darwin.go`, `service_stub.go` for other OSes |
| `lyrics/` | LRC parsing / fetching |
| `theme/` | Theme loading, default theme, `themes/` subfolder |
| `internal/` | Shared helpers not meant for import by plugins: `appdir`, `appmeta` (version), `browser`, `control`, `fileutil`, `httpclient`, `playback` (message types like `NextMsg`, `PrevMsg`), `resume`, `sshurl`, `tomlutil` |
| `cmd/` | Subcommand implementations that the CLI wires up (e.g. playlist subcommands) |
| `upgrade/` | Self-update logic for `cliamp upgrade` |
| `site/` | Static website for cliamp.stream — **keep synced with `docs/` on user-facing changes** |
| `docs/` | User-facing docs. Each feature has its own `.md`. Source of truth for keybindings, providers, plugin API |

### Runtime flow (read this before touching `main.go`)

1. `main()` builds the CLI (`buildApp()` in `commands.go`) and dispatches — most subcommands are thin IPC clients.
2. `run(overrides, positional)` is the TUI entry path:
   - Load config → apply CLI overrides.
   - Instantiate providers conditionally (radio + local always; navidrome/plex/jellyfin/spotify/ytmusic when configured).
   - Resolve positional args into tracks via `resolve/`.
   - Construct the `player.Player` (platform-specific audio device picked at compile time via build tags).
   - Build the Bubbletea `model.Model` with the player, playlist, providers, themes, and the Lua plugin manager.
   - Wire Lua state/control/UI providers (so plugins can read state and post `prog.Send(...)` messages).
   - Start the IPC server on a Unix socket.
   - Hand off to the media-control service (dbus / NowPlaying) which owns the event loop.

### Provider contract

All providers implement the interfaces in `provider/interfaces.go` (Browse, Tracks, Search where relevant). When adding a provider, add a package under `external/<name>/`, then register it in `main.go` behind a config check. Follow existing providers (Navidrome / Plex / Jellyfin) as templates — they share a lot of shape.

### Plugin surface


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bjarneo/cliamp](https://github.com/bjarneo/cliamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
