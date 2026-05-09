---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Gomoku-C** is a terminal-based Gomoku (Five-in-a-Row) game written in C for Linux/WSL2. The project features single-player (vs AI), multiplayer (LAN-based), spectator mode, and replay functionality with a sophisticated ncurses-based UI.

**Target Platform**: Linux/WSL2
**UI**: Terminal-based (ncurses with wide character support), minimum terminal size 100x31
**Network**: TCP socket communication with custom binary protocol
**Build System**: CMake 3.10+

## Building and Running

### Standard Build

```bash
# Using build script (recommended)
./build_and_run.sh

# Clean build
./build_and_run.sh --clean

# Manual build
mkdir -p build && cd build
cmake ..
make
./gomoku-c
```

### Running Tests

```bash
# Run all tests
./run_tests.sh

# Individual tests
cd build
./test_board       # Board unit tests
./test_rules       # Rules unit tests
./test_ai          # AI engine tests
./test_phase2      # Phase 2 integration tests
./test_phase3_ui   # UI tests
./test_menu        # Menu tests
```

### Command Line Options

```bash
# Menu mode (default)
./gomoku-c

# Singleplay modes
./gomoku-c --singleplay --easy
./gomoku-c --singleplay --hard

# Multiplayer modes
./gomoku-c --multiplay-host
./gomoku-c --multiplay-client -ip 192.168.0.2
./gomoku-c --multiplay-client -ip 192.168.0.2 -port 9000

# Spectator mode (max 3 spectators)
./gomoku-c --spectator -ip 192.168.0.2 -port 7773
```

**Default port**: 7773

## Architecture Overview

This codebase follows a clean, modular architecture with clear separation between game logic, UI, and networking.

### Core Module Structure

```
src/
├── main.c                    # Entry point, mode routing
├── game/
│   ├── core/                 # Core game logic
│   │   ├── board.c/h            # 19x19 board state, move validation
│   │   ├── game_logic.c/h       # Win detection, five-in-a-row checking
│   │   ├── rules.c/h            # Renju rules (3-3, 4-4, overline forbidden)
│   │   └── turn_manager.c/h     # Turn control, timer management
│   ├── ai/                   # AI engines
│   │   └── ai_engine.c/h        # Easy (heuristic) + Hard (minimax/alpha-beta)
│   ├── mode/                 # Game modes
│   │   ├── singleplay/          # Player vs AI
│   │   ├── multiplay/           # Online PvP (host/client)
│   │   └── general/             # Shared mode utilities
│   └── feature/              # Game features
│       ├── game_logger.c/h      # Log moves to file (gomoku-YYYYMMDD-HHMM.omk)
│       ├── replay.c/h           # Replay playback system
│       ├── replay_viewer.c/h    # Replay UI
│       └── command.c/h          # Chat commands (/quit, /undo, /giveup)
├── ui/
│   ├── core/                 # UI infrastructure
│   │   ├── ui_manager.c/h       # Window layout manager (100x31 grid)
│   │   ├── theme.c/h            # Color theme system
│   │   ├── input_handler.c/h    # Unified input handling (keyboard + gamepad)
│   │   └── gamepad_input.c/h    # Gamepad support
│   ├── game/                 # In-game UI components
│   │   ├── board/               # Board rendering
│   │   ├── border/              # UI borders and decorations
│   │   ├── chat/                # Chat box UI
│   │   ├── info/                # Game info panel
│   │   └── log/                 # Move log UI
│   └── menu/                 # Menu system
│       ├── menu_ui.c/h          # Main menu
│       ├── modal_ui.c/h         # Modal dialogs
│       └── theme_selector_ui.c/h
├── network/
│   ├── core/                 # Network layer
│   │   ├── network.c/h          # TCP socket management, server/client
│   │   └── protocol.c/h         # Message serialization/deserialization
│   └── messages/             # Message definitions
│       ├── message.h            # Message union type
│       ├── message_types.h      # MessageType enum, MessageHeader
│       ├── connection.h         # Connection handshake messages
│       ├── game.h               # Game state messages
│       ├── chat.h               # Chat messages
│       └── spectator.h          # Spectator messages
└── utils/
    ├── config.c/h            # Configuration management
    ├── arg_parser.c/h        # CLI argument parsing
    └── terminal_check.c/h    # Terminal size validation
```

### Key Architectural Patterns

#### 1. UI Layout System (100x31 Grid)

The game uses a fixed 100x31 terminal layout defined in `ui/core/ui_manager.h`:

- **Board area** (left): 49x25 (columns 0-48, rows 0-24)
- **Info panel** (top-right): 52x3 (columns 48-99, rows 0-2)
- **Chat area** (middle-right): 52x21 (columns 48-99, rows 2-22)
- **Bottom panel**: 100x6 (columns 0-99, rows 24-29)
  - Last move display, current turn indicator
  - Timer, play time counter
  - System log

All UI components respect these boundaries. When modifying UI, ensure changes align with the layout in `src/ui/game/border/ingame_border.c`.

#### 2. Network Protocol Architecture

The network layer uses a custom binary protocol with a message-based architecture:

**Message Structure**:

```c
Message {
    MessageHeader header;  // type, sequence, timestamp, payload_size
    union payload {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whitedev7773/gomoku-c](https://github.com/whitedev7773/gomoku-c) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
