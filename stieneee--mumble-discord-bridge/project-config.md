---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mumble Discord Bridge is a Go application that bridges audio and text chat between Mumble and Discord voice channels. It allows users on both platforms to communicate seamlessly.

## Build & Development Commands

```bash
# Build (requires libopus-dev and libdave installed)
make                    # Build binary
make install-libdave    # Install libdave C++ library (DAVE E2EE)
make dev                # Build and run for development

# Testing
go test -race -v ./...                              # Run all tests with race detector
go test -race -count=10 ./internal/bridge ./pkg/bridgelib  # Stress test (CI runs this)
go test -race -coverprofile=coverage.out ./internal/bridge ./pkg/bridgelib  # With coverage

# Linting
make lint               # Run golangci-lint
make format             # Format code with go fmt

# Run with race detector (development)
make dev-race           # go run -race ./cmd/mumble-discord-bridge

# Clean build artifacts
make clean
```

**System dependencies**: Install `libopus-dev`, `cmake`, `g++`, and `pkg-config` (Ubuntu) before building. Run `make install-libdave` to install the libdave C++ library. Ensure `PKG_CONFIG_PATH` includes `~/.local/lib/pkgconfig` and `LD_LIBRARY_PATH` includes `~/.local/lib`.

## Architecture

### Package Structure

- **cmd/mumble-discord-bridge**: CLI entry point, configuration parsing, signal handling
- **internal/bridge**: Core bridge logic
- **internal/discord**: Discord client abstraction layer (disgo + DAVE E2EE)
- **pkg/bridgelib**: High-level API for multi-bridge deployments (used by patchcord.io)
- **pkg/logger**: Logger abstraction (supports bridge-specific logging)
- **pkg/sleepct**: Precision sleep utility for audio timing

### Core Components (internal/bridge)

**BridgeState** (`bridge.go`): Central state holder managing:
- Connection states (Discord, Mumble, overall)
- User tracking maps with dedicated mutexes
- Audio stream lifecycle
- Three operational modes: `auto`, `manual`, `constant`

**Concurrency Model**: Uses multiple mutexes with strict lock ordering:
```
BridgeMutex -> MumbleUsersMutex -> DiscordUsersMutex
```
Always acquire in this order to prevent deadlocks.

**Connection Managers** (`connection_manager.go`, `discord_connection_manager.go`, `mumble_connection_manager.go`):
- Implement resilient connections with automatic reconnection
- Emit ConnectionEvents through channels
- BaseConnectionManager provides common retry/health check logic

**Audio Duplex** (`discord_duplex.go`, `mumble_duplex.go`):
- Handle bidirectional audio streaming
- Use jitter buffers (configurable via TO_DISCORD_BUFFER/TO_MUMBLE_BUFFER)
- MumbleDuplex mixes multiple Mumble audio streams

**Event Handlers** (`discord_handlers.go`, `mumble_handlers.go`):
- DiscordListener: Handles voice state updates, message creation
- MumbleListener: Handles user changes, text messages, connection events

### BridgeLib (pkg/bridgelib)

**SharedDiscordClient** (`discord.go`): Wraps `discord.Client` (disgo+godave) to allow multiple bridge instances to share a single Discord session with health monitoring.

**BridgeInstance** (`bridge.go`): High-level wrapper that:
- Creates and manages internal BridgeState
- Handles mode-specific startup (auto/manual/constant)
- Provides EventDispatcher for external event consumption

**EventDispatcher** (`events.go`): Async event system with typed events (connection, user join/leave, bridge lifecycle).

### Bridge Modes

- **constant**: Always connected, auto-restarts on disconnect
- **auto**: Connects when users present on both sides, disconnects when empty
- **manual**: Controlled via chat commands (`!mumble-discord link/unlink`)

### Discord Client Abstraction (internal/discord)

The project uses [disgo](https://github.com/disgoorg/disgo) with [godave](https://github.com/disgoorg/godave) for Discord connectivity and DAVE E2EE support. The `internal/discord` package provides an abstraction layer:

- **`Client` interface** (`client.go`): Discord bot operations (connect, send messages, get users/guilds, create voice connections)
- **`VoiceConnection` interface** (`voice.go`): Voice channel operations (open, close, send/receive opus, speaking state)
- **`DisgoClient`** (`disgo_client.go`): Implementation using disgo with DAVE E2EE via `golibdave.NewSession`
- **`DisgoVoiceConnection`** (`disgo_voice.go`): Voice connection implementation using disgo's voice system

The abstraction ensures bridge code (`internal/bridge`) does not depend directly on any specific Discord library. The `godave` library requires `libdave` (a C++ shared library) installed on the system.

### Audio Architecture

See `docs/AUDIO-PACKET-FLOW.md` for detailed audio pipeline documentation including packet flow diagrams, buffer sizing, and metrics reference.

Key audio concepts:

- **RTP framing is handled by disgo's UDPConn.Write**: each call increments sequence by 1 and timestamp by 960 (20ms). The timestamp is a monotonic counter, not wall-clock aligned.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Stieneee/mumble-discord-bridge](https://github.com/Stieneee/mumble-discord-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
