---
trigger: always_on
description: > **Phase 5 ANM Integration** (2026-06-16): AnmPlayer + BorderPanel + BldAnmMap shipped.
---

# Agent Instructions — BattleTech CHI Rebuild

> **Phase 5 ANM Integration** (2026-06-16): AnmPlayer + BorderPanel + BldAnmMap shipped.
> Next priorities: Runtime ANM decompression, 135D dispatch tables, combat mech panel ANM.

## ⚠️ Documentation Priority Rule

**Documentation correctness has priority over implementation.** When you discover new information that contradicts existing docs:

1. Immediately search **all `.md` files** (`grep -rn` across `docs/`, `AGENTS.md`, `README.md`) for the incorrect claim.
2. Fix every occurrence before writing implementation code.
3. Commit doc fixes as a separate commit with message prefix `docs:`.
4. Also **update the GitHub wiki** pages that humans read — clone `git@github.com:velteyn/UnBattletech.wiki.git`, update relevant pages, commit and push. The wiki has `Home.md`, `Rebuild-Progress.md`, `Combat-System.md` (and any others found).

Do not implement new features if they are based on incorrect documentation assumptions — fix the docs first, then reassess the implementation plan.

## Build & Run

```bash
# Build C# (fast, catches compile errors)
cd BattleTechCHI && dotnet build

# Build + Run Godot editor (headless via xvfb)
cd BattleTechCHI && bash run.sh

# Build only (via script)
cd BattleTechCHI && bash build.sh

# Build Spice86-based emulator (UNBATTLETECH)
dotnet build UNBATTLETECH.csproj

# Run emulator (headless, verbose, with MCP on port 8081)
# Always kill stale ports first — port 20000 (HTTP API) holds over from prior runs
fuser -k 20000/tcp 8081/tcp 2>/dev/null
dotnet exec bin/Debug/net10.0/UNBATTLETECH.dll \
  --Exe "/home/velteyn/projects/Reversing/BATTLETECH_CHI/UNBTECH.exe" \
  --CDrive "/home/velteyn/projects/Reversing/BATTLETECH_CHI/" \
  --HeadlessMode Minimal --McpHttpPort 8081 --NoGui --VerboseLogs
```

## Godot Binary

Installed at `~/.local/bin/godot`. The Mono (C#) version requires a display
server — use `xvfb-run` on headless systems.

Project uses Godot 4.4 (.NET SDK 4.4.0, net8.0).

## Architecture

- **Core/** — GameLoop, StateManager, InputHandler, SaveManager
- **Data/** — GameEnums, GameState, DataModels, WeaponData, CipherDecoder
- **Maps/** — MapLoader, RleDecompressor, TileManager, WorldMapData, WorldMapView, LocalMapView
- **BLD/** — BldLoader, BldInterpreter, Fn1CD3Dispatcher
- **UI/** — EgaPalette, BorderPanel, StartupSequence, ShopScreen
- **Runner/** — Standalone test console app (not part of Godot build)

## Phase 4 Combat System (Implemented)

Combat lives in `BattleTechCHI/Scripts/Combat/` — 5 files, fully reworked from stubs:

### Files
- **CombatTypes.cs**: `MechState` (class with full 11-location armour/structure arrays, 10 ammo bins, heat, crits), `HitLocation` enum (11 body parts), `AmmoBin` struct, `ActionCode`/`CombatPhase`/`Direction8` enums
- **CombatState.cs**: Twin 12×24 fog grids, 24 unit slots, phase tracking, kill chain state. Methods: `InitFogGrids()`, `ClearFogForUnit()`, `ResetFogForUnit()`, `KillUnit()`, `AnyEnemiesAlive()`
- **CombatManager.cs**: Main loop — 12-phase state machine (Init→UnitLoop→Movement→Targeting→ToHit→Fire→PostFire→HeatDissipation→Complete). Maps to `ghidra_guess_1000_458C_1458C`. Features:
  - Movement: approach toward target if out of weapon range, 1 tile/tick, collision + bounds check, fog clearing on move
  - To-hit: full 2D6 formula (skill popcount + terrain + heat thresholds + story state penalty)
  - Damage: hit location table, armour→structure flow, critical hits, ammo decrement, kill chain
  - Cluster weapons: LRM/SRM per-missile damage via cluster hits table
  - Combat end: story state update, encounter rate reduction, world map restore
- **CombatResolver.cs**: RNG (24-bit LFSR), LoS (Bresenham ray-cast), to-hit computation, heat dissipation (pool→penalty→clear), hit location (front table via `RNG & 0x0A`), criticals (2D6≥8), ammo explosion, kill chain
- **AiController.cs**: AI targeting from story state properties (offsets 0x33-0x55), action code by distance (short≤3/medium≤6/long≤10), weapon selection by best range+damage with ammo check

### Key Combat Flows
1. **Entry**: BLD `SHOP_DISPATCH 0x2D` → `Fn1CD3Dispatcher.CombatEncounter()` → `GameMode.Combat` → `GameLoop.StartCombatEncounter()` → `CombatManager.StartCombat()`
2. **Round**: UnitLoop(0..23) → Movement → LoS check → ToHit(2D6) → Fire(damage) → next unit → HeatDissipation → stageCounter++ → next round
3. **End**: All enemies dead → victory → story state update → world map; stageCounter>20 → draw

## Fn1CD3Dispatcher Case Mapping (ALL 47 CASES IMPLEMENTED, verified vs decompiled C)

| Case | Name | Description |
|------|------|-------------|
| 0x01 | ENTER_BUILDING | Emits `BuildingEntered` event, sets up viewport |
| 0x02 | SHOW_GREETING | Entry text displayed via BLD text opcodes |
| 0x03 | EXIT_BUILDING | → WorldMap |
| 0x04 | SHOW_SHOP_ITEMS | Populate C618 slots (StateArray 0x18-0x1A) from shop strategy |
| 0x05 | BUY_ITEM_SINGLE | C618[sel]++ (price = old_type * 125 + 75), rotate to next type |
| 0x06 | SHOW_PLAYER_ITEMS | Find owned items, populate C618[0..2] for sale UI |
| 0x07 | BUY_ITEM_BULK | Buy via strategy (GetBuyPrice) |
| 0x08 | SELL_ITEM_BULK | Sell via strategy (GetSellPrice) |
| 0x09 | HOSPITAL_HEAL | Heal 4 units, cost 50cr |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [velteyn/UnBattletech](https://github.com/velteyn/UnBattletech) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
