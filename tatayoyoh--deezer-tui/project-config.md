---
trigger: always_on
description: Lightweight, self-contained Deezer music player in terminal. Rust, ~5-8 MB RAM, single static binary ~5 MB. No external media player (no mpv, vlc, ffmpeg).
---

# CLAUDE.md — Deezer TUI Player

## Project Overview

Lightweight, self-contained Deezer music player in terminal. Rust, ~5-8 MB RAM, single static binary ~5 MB. No external media player (no mpv, vlc, ffmpeg).

## Architecture Principles

### Strict Separation: Core Library vs TUI Frontend

Two crates in Cargo workspace:

```
deezer-tui/
├── Cargo.toml              # Workspace root
├── CLAUDE.md
├── README.md
├── .circleci/config.yml    # CI/CD (CircleCI)
├── crates/
│   ├── deezer-core/        # Library crate — all business logic, no UI
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── api/        # Deezer private API client
│   │       │   ├── mod.rs
│   │       │   ├── auth.rs       # ARL token + email/password login
│   │       │   ├── gateway.rs    # gw-light.php (getUserData, song.getData, etc.)
│   │       │   ├── media.rs      # media.deezer.com/v1/get_url
│   │       │   └── models.rs     # API response types (serde)
│   │       ├── decrypt.rs  # Blowfish CBC stripe decryption
│   │       ├── player/     # Audio playback engine
│   │       │   ├── mod.rs
│   │       │   ├── engine.rs     # rodio/cpal playback, queue management
│   │       │   ├── stream.rs     # HTTP progressive streaming + decrypt
│   │       │   └── state.rs      # Player state (playing, paused, position, volume)
│   │       └── config.rs   # Configuration (credentials, quality prefs)
│   │
│   └── deezer-tui/         # Binary crate — terminal UI only
│       ├── Cargo.toml
│       └── src/
│           ├── main.rs           # Entry point, fork logic (daemon/client)
│           ├── daemon.rs         # Background daemon (Unix socket server)
│           ├── client.rs         # TUI client (connects to daemon)
│           ├── protocol.rs       # IPC protocol (Command/ServerMessage enums)
│           ├── web_login.rs      # Browser-based OAuth login flow
│           ├── theme.rs          # Colors, styles, Deezer themes
│           └── ui/               # Ratatui rendering
│               ├── mod.rs
│               ├── login.rs      # Login screen
│               ├── player.rs     # Bottom player bar
│               ├── search.rs     # Search tab (multi-category)
│               ├── favorites.rs  # Favorites tab (multi-category)
│               ├── radio.rs      # Radios / Podcasts tab
│               ├── downloads.rs  # Downloads tab
│               ├── album_detail.rs # Album detail overlay
│               ├── popup.rs      # Context menus, modals, playlist picker
│               └── common.rs     # Shared widgets, Deezer logo pixel art
```

**Rule: `deezer-core` must NEVER depend on any TUI/UI crate.** Exposes clean async API any frontend (TUI, GUI, web, CLI) can consume.

**Rule: `deezer-tui` depends on `deezer-core`, handles rendering + input only.**

### Why This Separation Matters

- Swap TUI for native GUI (egui, iced, tauri) without touching audio/API logic
- Unit-test business logic independently from UI
- Potentially expose `deezer-core` as reusable crate

## Deezer API — How It Works

### Authentication

Deezer does NOT provide full-track streaming via public API (only 30s previews). Uses **private/undocumented API** (same as web player).

Three auth methods:
1. **ARL token** — 192-char cookie from logged-in browser session (easiest)
2. **Email/password** — MD5 hash + auth hash → obtain ARL programmatically
3. **Web browser login** — opens deezer.com in browser, intercepts ARL via URI handler (`web_login.rs`)

Auth flow:
1. Set ARL as cookie on `.deezer.com`
2. Call `deezer.getUserData` → get `api_token` (checkForm) + `license_token`
3. Tokens needed for all subsequent API calls

### Track Streaming Pipeline

```
1. song.getData(SNG_ID)        → TRACK_TOKEN, MD5_ORIGIN, metadata
2. media.deezer.com/v1/get_url → CDN streaming URL
   POST { license_token, track_tokens[], format: "MP3_128"|"MP3_320"|"FLAC" }
3. HTTP GET CDN URL             → encrypted audio stream
4. Blowfish CBC stripe decrypt  → raw audio (MP3 or FLAC)
5. symphonia decode             → PCM samples
6. rodio/cpal output            → speakers
```

### Audio Encryption (Blowfish CBC Stripe)

Custom (weak) encryption, NOT standard DRM:
- Algorithm: **Blowfish CBC** with fixed IV `\x00\x01\x02\x03\x04\x05\x06\x07`
- **Only every 3rd 2048-byte block** encrypted (blocks 0, 3, 6, 9…)
- Per-track key: `MD5(track_id)` XOR'd with master secret (16 bytes)
- Master secret extracted dynamically at runtime from Deezer's public web resources (NOT hardcoded)

Rust crates: `blowfish`, `cbc`, `md-5`

### Audio Quality Tiers

| Format   | Bitrate        | Requires        |
|----------|----------------|-----------------|
| MP3_128  | 128 kbps CBR   | Free account    |
| MP3_320  | 320 kbps CBR   | Premium         |
| FLAC     | ~1411 kbps     | HiFi / Premium+ |

Quality fallback: FLAC → MP3_320 → MP3_128 → MP3_64

## Cross-Platform Audio Output (No External Player)

Binary fully self-contained:

| Platform | Audio Backend | System Dependency          |
|----------|---------------|----------------------------|
| Linux    | ALSA          | `libasound2` (pre-installed on most distros) |
| macOS    | CoreAudio     | None (built into macOS)    |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tatayoyoh/deezer-tui](https://github.com/Tatayoyoh/deezer-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
