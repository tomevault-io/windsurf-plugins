---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Build & Run
```bash
crystal build src/rosegold.cr            # all versions + auto-detection (default)
```

By default all supported MC versions are compiled in and the protocol is auto-detected
via STATUS ping. To shrink the binary to a single version, require an exact-version
entrypoint (no flags, lives in the bot's own source) — this skips auto-detection:
```crystal
require "rosegold/26.2"   # only 26.2 (proto 776) compiled in; ~50% smaller binary
```
Both paths share one source of truth: `Rosegold::ENABLED_PROTOCOLS` in
`src/rosegold/versions.cr`.

Per-version game data comes from the [minecraft-data shard](https://github.com/rosegoldmc/minecraft-data.cr)
(`lib/minecraft-data/data/<version>/`), embedded at compile time via `Minecraft::Data.load`/
`.read_asset`. The data, its schema models, and the jar-based generator live in that repo.

### Testing
```bash
# Type-check without codegen (fast)
crystal build --no-codegen src/rosegold.cr

# Run all tests
crystal spec

# Run specific spec files
crystal spec spec/integration/interactions_spec.cr

# Run with environment variables for debugging
LOG_LEVEL=trace crystal spec spec/integration/interactions_spec.cr
LOG_PACKET=72 crystal spec  # Log specific packet types
```

### Code Quality
```bash
crystal tool format
bin/ameba
```

## Project Structure

```
src/rosegold/
├── client.cr              # Core client: connection, packet dispatch, tick loop
├── bot.cr                 # High-level bot API (movement, combat, inventory, chat)
├── chat_manager.cr        # Chat sending (signed/unsigned messages, commands)
├── spectate_server.cr     # Entry point for spectate server
├── control/
│   ├── physics.cr         # Movement, collision, gravity (817 lines)
│   ├── interactions.cr    # Block breaking, placing, eating, attacks
│   └── inventory.cr       # High-level pick/deposit/withdraw/throw
├── events/                # Event classes (Tick, HealthChanged, Died, etc.)
├── inventory/
│   ├── slot.cr            # Slot + 70+ DataComponent classes
│   ├── menu.cr            # Base menu with click/move logic
│   ├── menus/             # PlayerMenu, ChestMenu, CraftingMenu, FurnaceMenu, etc.
│   ├── container_handle.cr # Intent-level container operations
│   ├── recipe.cr          # RecipeRegistry + RecipeDisplayEntry
│   └── ...                # click_operation, slot_offsets, item_constants, etc.
├── packets/
│   ├── clientbound/       # 63 clientbound packet classes
│   ├── serverbound/       # 40 serverbound packet classes
│   └── protocol_mapping.cr # packet_ids macro for multi-version support
├── spectate/              # SpectateServer modules (9 files)
│   ├── server.cr          # TCP server, forwarded packet tables
│   ├── connection.cr      # Per-spectator connection, state machine
│   ├── play_session.cr    # Spectating state, world sync setup
│   ├── lobby.cr           # Lobby state (bot not connected yet)
│   ├── packet_relay.cr    # Raw packet forwarding with entity ID remapping
│   └── ...                # handshake, configuration, world_sync, monitoring
├── world/
│   ├── dimension.cr       # Chunk storage, block lookups, entity tracking
│   ├── player.cr          # Position, health, effects, AABB constants
│   ├── chunk.cr           # Column of sections + block entities
│   ├── section.cr         # 16x16x16 paletted block/biome storage
│   ├── entity.cr          # Entity struct with metadata, passengers
│   ├── mcdata.cr          # Facade over the minecraft-data shard (per-protocol data)
│   └── ...                # vec3, aabb, look, player_list, heightmap
└── models/
    ├── event_emitter.cr   # Pub/sub: on/off/once/wait_for/emit_event
    ├── text_component.cr  # Rich text (NBT-based, MC 1.21+)
    └── block.cr           # Block properties, break speed calculation
```

## Architecture Overview

### Layered Design

```
Bot (high-level DSL: move_to, dig, craft, chat)
 └── Client (connection, packets, state, tick loop)
      ├── Physics (movement, collision, gravity)
      ├── Interactions (digging, placing, eating, attacks)
      ├── Inventory (pick, deposit, withdraw)
      ├── Dimension (chunks, entities, block state)
      ├── Player (position, health, effects)
      └── ChatManager (send messages/commands)
```

**Client** manages the TCP connection, reads/dispatches packets, runs the game tick loop (50ms = 20 TPS), and holds all game state.

**Bot** is a thin wrapper that subscribes to Client events, re-emits them, and provides the user-facing API.

### Connection Lifecycle

HANDSHAKING → LOGIN → CONFIGURATION → PLAY (→ re-CONFIGURATION → PLAY)

Protocol version auto-detected via STATUS ping (among the versions compiled in). Supports a contiguous range: 772 (1.21.8), 773 (1.21.9/1.21.10), 774 (1.21.11), 775 (26.1), 776 (26.2). Compression enabled during LOGIN via SetCompression. Which versions are compiled in is controlled at build time — see "Build & Run".

### Packet System

~109 packet classes (70 clientbound + 39 serverbound). All packets extend `Rosegold::Event`, so they flow through the event system. Each packet defines:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RosegoldMC/rosegold.cr](https://github.com/RosegoldMC/rosegold.cr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
