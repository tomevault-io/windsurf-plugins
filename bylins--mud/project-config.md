---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **Bylins MUD** - a Russian-language Multi-User Dungeon game server based on CircleMUD/DikuMUD. The codebase is ~272K lines of C++20 code with extensive Russian comments and variable names. It implements a complete multiplayer text-based RPG with combat, magic, crafting, quests, and scripting systems.

## Build Commands

### Initial Setup (Ubuntu/WSL)
```bash
sudo apt update && sudo apt upgrade
sudo apt install build-essential make libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext unzip cmake gdb libgtest-dev zlib1g-dev
git clone --recurse-submodules https://github.com/bylins/mud
cd mud
cp -n -r lib.template/* lib
```

### Если репозиторий уже склонирован без submodules

```bash
git submodule update --init --recurse-submodules
```

### Standard Build (without tests)
```bash
mkdir build
cd build
cmake -DBUILD_TESTS=OFF -DCMAKE_BUILD_TYPE=Test ..
make -j$(($(nproc)/2))
cd ..
./build/circle 4000  # Start server on port 4000
```

### Build with Tests
```bash
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Test ..
make tests -j$(($(nproc)/2))
./tests/tests  # Run all tests
```

**Important:** Always use `-j$(($(nproc)/2))` for parallel builds to avoid overloading the system.

### Build Types
- **Release** - Optimized production build (-O0 with debug symbols, -rdynamic, -Wall)
- **Debug** - Debug build with ASAN (-fsanitize=address, NO _GLIBCXX_DEBUG due to yaml-cpp ABI compatibility)
- **Test** - Test build with optimizations (-O3, -DTEST_BUILD, -DNOCRYPT)
- **FastTest** - Fast test build (-Ofast, -DTEST_BUILD)

### Docker Build & Run
```bash
docker build -t mud-server --build-arg BUILD_TYPE=Test .
docker run -d -p 4000:4000 -e MUD_PORT=4000 -v ./lib:/mud/lib --name mud mud-server
docker stop mud
```

### Running the Server

**CRITICAL**: ALL binaries (circle, tests, converters, etc.) must ALWAYS be run from their build directory, NEVER from the source directory. Running binaries from the source directory creates runtime files (data/, misc/, etc.) in the wrong location and pollutes the source tree.

**CRITICAL**: Build directories must ALWAYS be out-of-source (outside the source tree). Never run cmake or binaries from within the src/ directory or repository root.

**CRITICAL**: CMake automatically creates test world `small/` in the build directory. All world data and configs are in `small/` directory itself, NOT in `small/lib/`. The `lib/` subdirectory DOES NOT EXIST in cmake-generated worlds. All paths in configuration.xml are relative to the `small/` directory.

**Correct usage**:
```bash
cd build_debug       # Or build_yaml, build_sqlite, etc.
./circle [-W] -d small <port>
```

**Parameters**:
- `-W` - Enable world checksum calculation (optional)
- `-d <world_directory>` - Specify world data directory (e.g., `small`)
- `<port>` - Port number to listen on (e.g., `4000`)

**Example**:
```bash
cd build_debug
./circle -d small 4000
```

**World Structure (cmake-generated):**
```
build_debug/
├── circle           # Binary
└── small/           # Test world (created by cmake)
    ├── misc/
    │   └── configuration.xml
    ├── world/       # Zone files
    ├── etc/         # Additional configs
    └── admin_api.sock  # Unix socket (if Admin API enabled)
```

**NOTE**: Do NOT confuse with repository's `lib/` directory - that is completely separate and used only for production deployments.

### Running Tests
```bash
# Run all tests
cd build
./tests/tests

# Run with CTest
make checks  # or: ctest -V
```

## Code Architecture

### High-Level Organization

The codebase is organized into three major sections:

**`src/engine/`** - Core MUD infrastructure (network, database, scripting VM, entities)
- `boot/` - Initialization and data file loading
- `core/` - Game loop (25Hz heartbeat), communication layer, command processing
- `db/` - World persistence, player save/load, InfluxDB stats integration
- `entities/` - Core entity classes: `CharData`, `ObjData`, `RoomData`, `Zone`
- `network/` - Socket handling, telnet protocol, MSDP, MCCP compression
- `olc/` - Online creation editors (Oedit, Redit, Medit, Zedit, Trigedit)
- `scripting/` - DG Scripts trigger system (20+ trigger types for mobs/objects/rooms)
- `structs/` - Data structures (compact trie, radix trie, blocking queue, flags)
- `ui/` - User interface and command system
  - `cmd/` - 100+ player commands (do_*.cpp/h)
  - `cmd_god/` - Admin/immortal commands

**`src/gameplay/`** - Game-specific mechanics (17 subsystems)
- `abilities/` - Feats and talents system
- `affects/` - Temporary status effects (buffs/debuffs)
- `ai/` - Mobile AI, pathfinding, spec_procs
- `clans/` - Guild/house system
- `classes/` - Character and mob classes with progression
- `communication/` - Boards, mail, messaging, social
- `core/` - Base stats, character generation, constants
- `crafting/` - Item creation, jewelry, mining, frying
- `economics/` - Shops, auction, exchange, currencies
- `fight/` - Combat system with hit calculation, penalties, assists
- `magic/` - 108+ spells, spell effects, magic items/rooms
- `mechanics/` - Core gameplay (corpses, equipment, doors, mounts, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bylins) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
