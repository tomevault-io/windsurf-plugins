---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Running
```bash
# Build in release mode (optimized)
cargo build --release

# Run the server (requires DATABASE_PASSWORD environment variable)
DATABASE_PASSWORD=ragnarok cargo run --package server --bin server

# Run with visual debugger (requires visual_debugger feature)
cargo run --package server --bin server --features visual_debugger
```

### Testing
```bash
# Run all tests
cargo test --release

# Run integration tests (requires running PostgreSQL database)
cargo test --features integration_tests

# Run unit tests only
cargo test --features unit_tests
```

## Architecture Overview

This is a Ragnarok Online MMORPG server implementation written in Rust that combines the traditional Login/Char/Map server architecture into a single unified server. The architecture is built around message-passing and event-driven design patterns.

This project focus exclusively on "pre-re" (or "pre renewal") version of the game.

### Key Architectural Principles
- **Message Passing**: Uses message passing instead of shared memory with locks to avoid deadlocks and ensure thread safety
- **Event-Driven**: State changes occur through queued events processed by dedicated loops
- **Service Layer**: Business logic is separated into service layers
- **Repository Pattern**: Data access is abstracted through repository interfaces

### Core Components

#### Game Loop System
- Main game loop runs at fixed intervals (40ms tick rate)
- Events are queued for future ticks and processed sequentially
- State updates trigger persistence events and client notifications

#### Thread Architecture
1. **Game Loop Thread**: Processes game events and state updates
2. **Map Instance Threads**: Each map instance runs its own event loop for scalability
3. **Persistence Thread**: Handles all database write operations
4. **Client Notification Thread**: Sends packets to connected clients

#### Major Modules
- `server/`: Core server implementation
- `server/src/server/boot/`: Server initialization (map loading, script compilation, etc.)
- `server/src/server/service/`: Business logic layer (character, battle, inventory, etc.)
- `server/src/server/repository/`: Data access layer with PostgreSQL integration
- `server/src/server/request_handler/`: Packet handling controllers
- `server/src/server/script/`: Integration with rAthena script virtual machine
- `server/src/server/state/`: Game state management (characters, maps, mobs)
- `server/src/server/mod.rs`: Implementation of server threads
- `server/src/server/game_loop.rs`: Implementation of the main game loop, latency of operation within the game loop should be low (<20ms) or server will lag. There is only one loop for the whole server. it handles action made by player
- `server/src/server/map_instance_loop.rs`: Implementation of map instance loop, responsible to handle action on a specific map, like interaction with MOB or NPC. There is one loop per map instance thread.
- `server/src/server/persistence.rs`: Implementation of an event loop for all database interaction which can be defered.
- `server/src/server/request_handler/mod.rs`: A function calling packet parser then by downcasting reference to the packet implementation route the request to the right function to handle it.
- `server/src/server/model/events/game_event.rs`: Contains enumeration of game event, that are handled by the game loop. It is used for message passing between request handler thread and game loop.
- `server/src/server/model/events/client_notification.rs`: Enumeration for sending packet to the client
- `server/src/server/state/server.rs`: Access to server state, access should only be done from game loop, state can be accessed for mutability in an unsafe way, thus it is mandatory to access it only from main game loop
- `/src/repository/`: data access layer, this is the implementation of database interaction using `sqlx`, we write mainly raw SQL with prepared statement.
- `/src/tests/`: unit and integration test of the server
- `lib/`: crate for specific logic implementation
  - `lib/configuration`: Structure for configuration the server. This is where configuration entry should be added 
  - `lib/models`: Structures shared accross crates
  - `lib/packets`: Structures of all packets exchanged between client and server
  - `lib/skills`: Structures containing implementation of all class skills
### Configuration and Data
- `config.json`: Main server configuration (copy from `config.template.json`)
- PostgreSQL database for persistent data (accounts, characters, etc.)

### Sending packet to the client
- All services have an instance of `client_notification_sender: SyncSender<Notification>,` in their structure
- `Notification` is an enum implemented at `server/src/server/model/events/client_notification.rs`
- `Notification` inner structures contains raw packet data Vec<u8> which come from any packets structure `raw` field
- `server/src/server/mod.rs` contains implementation of `client_notification_thread` which route and send packet to the right socket(s) 

### Server state


## Files to read or to avoid
### Read

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nmeylan/rust-ro](https://github.com/nmeylan/rust-ro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
