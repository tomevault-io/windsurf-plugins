---
trigger: always_on
description: Guidelines for AI agents working in this repository.
---

# AGENTS.md

Guidelines for AI agents working in this repository.

**Maintenance rule:** Any time you add or modify a feature — new lyric provider, new config field, new WebSocket message type, changed API endpoint, changed behavior — you must update the relevant sections of this file in the same change. Do not defer documentation to a follow-up commit.

## Project Overview

**Lyrica** is a lightweight Linux desktop lyrics display application. It consists of:

- A **Rust backend** daemon that discovers MPRIS2 music players via D-Bus, fetches lyrics from multiple sources, synchronizes lyric lines in real time, and exposes results over a WebSocket/HTTP server on `127.0.0.1:15650`.
- A **KDE Plasma 6 Plasmoid** (QML) that connects to the backend and renders the current lyric line on the panel.
- An **OBS Studio plugin** (Python) that similarly consumes the WebSocket feed.

## Repository Layout

```
src/                      # Rust backend
  main.rs                 # Entry point: HTTP server, MPRIS loop spawn
  config.rs               # Config struct and persistence
  state.rs                # Shared app state + broadcast channel
  messages.rs             # ChannelMessage enum (WebSocket payloads)
  lyric_parser.rs         # LRC timestamp parser
  helpers.rs              # Utility trait extensions
  websocket.rs            # Actix WebSocket actor
  player/                 # MPRIS2 player lifecycle
    mpris_loop.rs         # Top-level event loop
    player_discovery.rs   # D-Bus NameOwnerChanged listener
    player_observation.rs # Property change watcher
    lyric_session.rs      # Real-time lyric synchronization (SessionManager)
    mpris_metadata.rs     # Metadata extraction helpers
    dbus_proxies.rs       # zbus D-Bus proxy definitions
  lyric_cache.rs          # Disk lyric cache (read/write/cleanup)
  lyric_providers/        # Pluggable lyric sources
    mod.rs                # Provider registry, orchestration, fetch_netease_lyric helper
    file.rs               # Local tags (ID3/Vorbis) + LRC files
    mpris2_text.rs        # MPRIS2 Lyrics property fallback
    netease.rs            # NetEase Cloud Music search
    netease_trackid.rs    # NetEase via embedded track ID
    yesplaymusic.rs       # YesPlayMusic local API (port 10754)
    feeluown_netease.rs   # FeelUOwn integration
    splayer.rs            # SPlayer WebSocket (port 25885)
  web_routes/             # Actix HTTP handlers
    config.rs             # GET /config, POST /config/update
    test_page.rs          # Debug page at /test
frontend/
  kde/                    # Plasma 6 Plasmoid
    metadata.json         # Plugin manifest (version auto-injected by build script)
    contents/ui/          # QML files (main.qml, configBackend.qml, configFrontend.qml)
    config/               # KConfig XML schema
    translate/            # .po translation files (zh_CN, zh_TW)
  obs_studio/             # OBS Studio Python plugin
  build_kde_plasmoid.sh   # Builds .plasmoid package
docs/                     # User-facing documentation
assets/                   # Logos and screenshots
Cargo.toml / Cargo.lock   # Rust package manifest and lockfile
```

## Language and Toolchain

| Component | Language | Toolchain |
|-----------|----------|-----------|
| Backend | Rust 2024 edition | `cargo build --release` |
| Plasmoid | QML (Qt 6 / KDE Plasma 6) | bundled by `build_kde_plasmoid.sh` |
| OBS plugin | Python 3 | no build step |
| Build scripts | Bash | run directly |

Build the backend:
```sh
cargo build --release
```

Build the Plasma plasmoid package (`.plasmoid`):
```sh
cd frontend && bash build_kde_plasmoid.sh
```

## Architecture Notes

### Communication Flow

```
Music player (e.g. Elisa, Spotify)
  └─ D-Bus / MPRIS2
       └─ player/ module  →  lyric_providers/  →  state.rs
                                                      └─ broadcast channel
                                                           └─ websocket.rs  →  plasmoid / OBS
```

1. `mpris_loop` discovers the top-most MPRIS2 player.
2. `player_observation` watches `PropertiesChanged` on the player and fires metadata/seek/rate events.
3. `lyric_session::SessionManager` schedules wake-up timers based on LRC timestamps, playback position, and rate.
4. On each wake-up, the current lyric line (and optional translated line) is sent to `state.rs` which re-broadcasts via the Tokio broadcast channel.
5. Every connected WebSocket client (`websocket.rs`) receives the update.

### Lyric Provider Chain

Providers are tried in order; the first successful response wins. Order is defined in `lyric_providers/mod.rs`. Providers can be enabled/disabled at runtime via `Config.enabled_lyric_providers`.

Current providers (in default priority order):
1. `Mpris2Text` — MPRIS2 `xesam:asText` property
2. `File` — local ID3/Vorbis tags, then `~/Music/lrc/<title>.lrc`
3. `YesPlayMusic` — local YesPlayMusic API (localhost:10754)
4. `NeteaseTrackID` — embedded NetEase track ID in MPRIS metadata
5. `SPlayer` — SPlayer WebSocket (localhost:25885)
6. `FeelUOwnNetease` — FeelUOwn Python integration
7. `Netease` — online NetEase Cloud Music search


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chiyuki0325/lyrica](https://github.com/chiyuki0325/lyrica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
