---
trigger: always_on
description: This file guides Claude Code when working in this repository.
---

# CLAUDE.md

This file guides Claude Code when working in this repository.

## Project Overview

`spotify-player` is a terminal Spotify client (requires Spotify Premium) written in Rust. It supports playback control, Spotify Connect, direct streaming via librespot, synced lyrics, desktop notifications, OS media controls, album art rendering, a daemon mode, and a full CLI interface.

## Architecture

### Key modules in `spotify_player/src/`

| Module                        | Responsibility                                                              |
| ----------------------------- | --------------------------------------------------------------------------- |
| `main.rs`                     | Entry point; wires threads/tasks; CLI arg parsing; logging init             |
| `state/`                      | Shared app state (`Arc<State>`): UI, player data, library caches, queue     |
| `state/model.rs`              | Core domain types: `Track`, `Album`, `Artist`, `Playlist`, `Playback`, etc. |
| `state/player.rs`             | `PlayerState`: current playback, devices, queue, progress estimation        |
| `state/data.rs`               | `AppData`: user library, TTL memory caches, file-cache persistence          |
| `state/ui/`                   | `UIState`: page history stack, popup state, key buffer, count prefix        |
| `client/mod.rs`               | `AppClient`: Spotify API calls, session management                          |
| `client/request.rs`           | `ClientRequest` / `PlayerRequest` enums (async message types)               |
| `client/handlers.rs`          | Tokio task: receives `ClientRequest`, dispatches API calls                  |
| `config/mod.rs`               | `Configs` loaded once into a `OnceLock`; read via `config::get_config()`    |
| `config/keymap.rs`            | Default keybindings and key sequence lookup                                 |
| `config/theme.rs`             | Theme definitions and style resolution                                      |
| `command.rs`                  | `Command` enum (all TUI commands), `Action` / `CommandOrAction`             |
| `key.rs`                      | `Key` / `KeySequence` types; vim-style key parsing (`C-x`, `M-x`)           |
| `event/mod.rs`                | crossterm input loop; routes events to page or popup handler                |
| `event/{page,popup}.rs`       | Key event dispatch per page / popup overlay                                 |
| `ui/mod.rs`                   | ratatui render loop; main layout dispatch                                   |
| `ui/{page,playback,popup}.rs` | Render functions for pages, playback bar, popups                            |
| `ui/streaming.rs`             | FFT audio visualizer: `VisualizationSink`, `VisBands`, bar chart            |
| `streaming.rs`                | librespot connection + audio backend setup (feature-gated)                  |
| `cli/`                        | Unix socket server and client for inter-process CLI commands                |
| `auth.rs`                     | OAuth scopes and librespot credential/session building                      |
| `media_control.rs`            | OS media key integration via `souvlaki` (feature-gated)                     |

### Concurrency model

Multiple OS threads communicate via `flume` channels and `Arc<State>`:

- **Event thread** — blocking `crossterm::event::read()` loop
- **UI thread** — poll-based ratatui render loop
- **Tokio runtime** — async tasks: client handler, socket listener, streaming
- **Player-event watcher** — polls librespot playback state, sends `ClientRequest`s
- **Media-control thread** — OS media key events (feature-gated)

Event/UI threads never call async functions directly. They send a `ClientRequest` over a `flume` channel; the async handler updates `Arc<State>`.

### Feature flags

| Feature                                                                                                                                 | Effect                                                   |
| --------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| `streaming`                                                                                                                             | librespot playback, Spotify Connect, audio visualization |
| `rodio-backend`                                                                                                                         | Default audio sink (rodio)                               |
| `alsa-backend`, `pulseaudio-backend`, `portaudio-backend`, `jackaudio-backend`, `rodiojack-backend`, `sdl-backend`, `gstreamer-backend` | Alternative audio sinks                                  |
| `media-control`                                                                                                                         | OS media key integration                                 |
| `image`                                                                                                                                 | Album art rendering                                      |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aome510/spotify-player](https://github.com/aome510/spotify-player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
