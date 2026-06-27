---
trigger: always_on
description: This is a **racing simulator HUD plugin** for PiBoSo racing games (MX Bikes, GP Bikes, WRS, KRP). It's a DLL plugin written in C++ using each game's proprietary API, with a shared core that works across all supported games.
---

# AI Development Context for MXBMRP3

## Read This First

This is a **racing simulator HUD plugin** for PiBoSo racing games (MX Bikes, GP Bikes, WRS, KRP). It's a DLL plugin written in C++ using each game's proprietary API, with a shared core that works across all supported games.

**For deep technical details:** See [`ARCHITECTURE.md`](ARCHITECTURE.md) (comprehensive documentation with mermaid diagrams, component descriptions, dependency graphs, multi-game architecture). This file is a quick-start guide.

## Quick Architecture

```
Game Engine (MX Bikes / GP Bikes / WRS / KRP)
    ↓ (callbacks via plugin API)
mxb_api.cpp / gpb_api.cpp (per-game DLL exports)
    ↓ (converts to unified types via adapters)
PluginManager (receives unified types only)
    ↓
PluginData (singleton - caches all game state)
    ↓ (notifies on data changes)
HudManager (singleton - owns all HUD instances)
    ↓
Individual HUDs (IdealLap, Standings, Map, etc.)
    ↓ (build render primitives)
Game Engine (renders quads/strings)

PluginData ──(notifies on data changes)──→ HttpServer
    ↓ (builds JSON snapshot on game thread)
SSE stream → Web Overlay (browser/OBS)
```

**Key Singletons:**
- `PluginData` - Central game state cache, change detection
- `HudManager` - HUD lifecycle, owns all HUD instances
- `SettingsManager` - Save/load HUD configurations
- `InputManager` - Mouse and keyboard input
- `XInputReader` - Controller state and rumble effects
- `RumbleProfileManager` - Per-bike rumble profiles stored in JSON
- `StatsManager` - Unified stats, personal bests, odometers in a single JSON file
- `FmxManager` - FMX trick detection state machine, scoring, chain system
- `AssetManager` - Dynamic discovery of fonts, textures, icons from subdirectories
- `FontConfig` - User-configurable font categories (Title, Normal, Strong, Marker, Small)
- `ColorConfig` - User-configurable color palette
- `HttpServer` - Embedded HTTP server with SSE streaming for web overlays (OBS)

## Multi-Game Support

The plugin supports multiple PiBoSo games from a single codebase:

| Game | Config | Output | Status |
|------|--------|--------|--------|
| MX Bikes | `MXB-Release` | `mxbmrp3.dlo` | ✅ Full support |
| GP Bikes | `GPB-Release` | `mxbmrp3_gpb.dlo` | ✅ Core features |
| Kart Racing Pro | `KRP-Release` | `mxbmrp3_krp.dlo` | ✅ Core features (no FMX) |
| WRS | - | `wrsmrp3.dlo` | ⏳ Stubbed |

**Translation Layer:**
- `game/unified_types.h` - Game-agnostic data structures (`Unified::` namespace)
- `game/game_config.h` - Compile-time game selection, feature macros
- `game/adapters/*_adapter.h` - Convert game structs → unified types
- `vendor/piboso/*_api.cpp` - Per-game DLL exports

## Build & Test

**⚠️ IMPORTANT - Build Environment:**
- This is a **Windows-only** Visual Studio project
- **DO NOT attempt to build in Linux/WSL environments** - it will fail
- Claude Code often runs in Linux - you cannot build this project there
- Instead: read code, make edits, commit changes, and let Windows users build

**⚠️ IMPORTANT - Shell Commands:**
- The user runs on **Windows**, not Linux
- When providing shell commands for the user to run, use Windows syntax:
  - Use `&` instead of `&&` for chaining commands (or provide separate commands)
  - Use backslashes `\` for paths, or forward slashes `/` (git accepts both)
  - Example: `git fetch origin & git reset --hard origin/branch-name`

**Build Instructions (Windows only):**
- **Build**: Open `mxbmrp3.sln` in Visual Studio 2022 (C++17, v143 toolset)
- **Platform**: x64 only (all PiBoSo games are 64-bit)
- **Configurations**:
  - `All-Release` / `All-Debug` → builds MXB + GPB + KRP sequentially via the `build_all` meta-project (default in the dropdown)
  - `MXB-Debug` / `MXB-Release` → `build/MXB-Release/mxbmrp3.dlo`
  - `GPB-Debug` / `GPB-Release` → `build/GPB-Release/mxbmrp3_gpb.dlo`
  - `KRP-Debug` / `KRP-Release` → `build/KRP-Release/mxbmrp3_krp.dlo`
- **Deploy**: Copy `.dlo` to game's `plugins/` folder
- **Debug**: Use Debug configuration (enables DEBUG_INFO macros automatically)

## Important Patterns & Constraints

### Performance Target: 240fps
The plugin must run efficiently at **240fps** (4.17ms frame budget). Many competitive players use high refresh rate monitors. Avoid per-frame allocations, unnecessary string operations, and complex calculations in hot paths like `Draw()` and `RunTelemetry()`.

### DO:
- Use RAII (smart pointers, no raw `new`/`delete`)
- Use safe string functions (`strncpy_s`, `snprintf`)
- Add exception handling for file I/O
- Use `DEBUG_INFO_F()` for logging (not `printf`)
- Check for existing patterns before adding new code
- Wrap new DLL exports in `API_GUARD_CATCH("ExportName")` (see `vendor/piboso/api_guard.h`); uncaught exceptions across the boundary crash the host game
- Wrap new `std::thread` function bodies in a top-level try/catch; uncaught throws in threads call `std::terminate()`
- Exception-guard **every** INI parse site — hand-editing the INI (`auto_save` off + RELOAD_CONFIG) is a supported workflow, and one naked `std::stoul` aborts the whole settings load (the `parseColorHex` base-section bug)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thomas4f/mxbmrp3](https://github.com/thomas4f/mxbmrp3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
