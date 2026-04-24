---
trigger: always_on
description: Sendspin CLI is a synchronized audio player client for the [Sendspin Protocol](https://github.com/Sendspin/website/blob/main/src/spec.md). It connects to Sendspin servers via WebSocket, receives time-synchronized audio streams, and plays them back with precise timing to enable multi-room synchronized audio.
---

## Project Overview

Sendspin CLI is a synchronized audio player client for the [Sendspin Protocol](https://github.com/Sendspin/website/blob/main/src/spec.md). It connects to Sendspin servers via WebSocket, receives time-synchronized audio streams, and plays them back with precise timing to enable multi-room synchronized audio.

**Note**: If uncertain about how something in Sendspin is supposed to work, fetch and refer to the [protocol specification](https://github.com/Sendspin/website/blob/main/src/spec.md) for authoritative implementation details.

## Architecture

```
                            cli.py (main entry)
                                    │
                    ┌───────────────┼───────────────┐
                    │                               │
                    ▼                               ▼
              ┌──────────┐                    ┌──────────┐
              │   tui/   │                    │ daemon/  │
              └──────────┘                    └──────────┘
                    │                               │
        ┌───────────┼───────────┐                   │
        │           │           │                   │
        ▼           ▼           ▼                   ▼
┌────────────┐ ┌─────────┐ ┌─────────┐      ┌─────────────┐
│ keyboard   │ │   ui    │ │   app   │      │   daemon    │
│            │ │         │ │         │      │             │
│ - Keys     │ │ - TUI   │ │ - State │      │ - Headless  │
│ - Commands │ │ - Panel │ │ - Tasks │      │ - Signals   │
└────────────┘ └─────────┘ └─────────┘      └─────────────┘
        │           │           │                   │
        └───────────┼───────────┴───────────────────┘
                    │
        ┌───────────┼───────────────────┐
        │           │                   │
        ▼           ▼                   ▼
┌─────────────┐ ┌───────────────┐ ┌──────────────┐
│ audio.py    │ │ discovery.py  │ │ aiosendspin  │
│             │ │               │ │ (external)   │
│ - Playback  │ │ - mDNS        │ │              │
│ - Time sync │ │ - Servers     │ │ - WebSocket  │
└─────────────┘ └───────────────┘ └──────────────┘
```

## File Responsibilities

### `README.md`
Project documentation, installation instructions, usage guide (including command line arguments), configuration options.

### `sendspin/cli.py`
Main entry point and orchestrator. Responsibilities:
- **Argument parsing**: Commands (`serve`, `daemon`), flags (`--url`, `--name`, `--id`, `--audio-device`, `--static-delay-ms`)
- **Command routing**: Routes to appropriate mode (TUI app, daemon, or server)
- **Device enumeration**: Lists audio devices and servers via `audio-devices list` and `servers list` subcommands
- **Backward compatibility**: Handles deprecated `--headless` flag with warning

### `sendspin/tui/` (TUI Mode Package)
Interactive terminal UI mode with keyboard control.

#### `sendspin/tui/app.py`
TUI mode application logic. Responsibilities:
- **Service discovery**: mDNS discovery of Sendspin servers
- **Connection management**: `ConnectionManager` handles reconnection with exponential backoff
- **State management**: `AppState` dataclass mirrors server state (playback, metadata, volume)
- **Audio stream handling**: `AudioStreamHandler` bridges between client and `AudioPlayer`
- **Event callbacks**: Routes server messages to appropriate handlers (metadata, group updates, commands)
- **UI coordination**: Manages `SendspinUI` and keyboard input loop

#### `sendspin/tui/keyboard.py`
Keyboard input handling for interactive control. Responsibilities:
- **Key capture**: Uses `readchar` library for single-keypress detection
- **Command parsing**: `CommandHandler` class parses and executes commands
- **Media commands**: play, pause, stop, next, previous, shuffle, repeat modes, switch group
- **Volume control**: Group volume (`vol+`/`vol-`/`mute`) and player volume (`pvol+`/`pvol-`/`pmute`)
- **Delay adjustment**: Real-time static delay adjustment via `delay` command
- **Keyboard shortcuts**: Arrow keys (prev/next/volume), space (toggle), m (mute), g (switch group), q (quit)

#### `sendspin/tui/ui.py`
Rich-based terminal UI for visual feedback. Responsibilities:
- **Now Playing panel**: Track title, artist, album with playback shortcuts
- **Volume panel**: Group and player volume with mute indicators
- **Progress bar**: Real-time track progress with interpolation during playback
- **Status line**: Connection status and quit shortcut
- **Shortcut highlighting**: Visual feedback when shortcuts are pressed (0.15s highlight)
- **State management**: `UIState` dataclass for all display state

### `sendspin/daemon/` (Daemon Mode Package)
Headless operation mode for background services.

#### `sendspin/daemon/daemon.py`
Daemon mode for headless operation (via `sendspin daemon` or deprecated `--headless`). Responsibilities:
- **Headless operation**: Runs without UI or keyboard input
- **Service discovery**: Same mDNS discovery as TUI mode
- **Connection management**: Reuses `ConnectionManager` from `tui/app.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sendspin/sendspin-cli](https://github.com/Sendspin/sendspin-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
