---
trigger: always_on
description: External memory-reading **map/radar overlay for Path of Exile 2**. .NET 10, Windows, x64 only.
---

# POE2GPS — Contributor Guide

External memory-reading **map/radar overlay for Path of Exile 2**. .NET 10, Windows, x64 only.
Reads game state out of process (no injection) and draws an overlay — read-only, no keystrokes or
input sent to the game. Forked from a PoE1 framework, since rewritten around the live PoE2 layout.

## Non-negotiable rules

**PoE2, not PoE1.** Offsets are PoE2-specific and drift with patches. Validated values live in
`Game/Poe2Offsets.cs` (marked `✓` when confirmed live); re-discover via the `POE2Radar.Research` probes.

**Stay external.** Memory access via `OpenProcess` + `ReadProcessMemory`. **Never** inject into the
PoE2 process — no DLL injection, no function hooking, no packet manipulation.

**No input emission.** The overlay never sends keystrokes or input to the game — the auto-flask
keystroke subsystem was removed on purpose (see README, `scripts/compliance-gate.ps1`). Hotkeys are
read-only and only fire while PoE2 is focused; the controller reader (`Input/XInputNative.cs`,
`Input/ControllerCycler.cs`) reads pad state for the HUD, it does not send input. CI's
`compliance-gate.ps1` fails the build on any input-emission or process-write Win32 symbol
(`SendInput`, `WriteProcessMemory`, `CreateRemoteThread`, …) in shipped source.

**Offset discovery lives in Research.** The overlay just reads; reverse-engineering/probes live in
`POE2Radar.Research`. When a patch breaks reads, run the Research probes, re-validate, commit — and
**bump the `supports PoE2 X.Y.Z` badge in `README.md`** to the patch you just validated, so the
at-a-glance currency signal stays accurate (if it lags the live patch, users see we're behind).

**Three-pillar layout.** Exactly three projects:
- `src/POE2Radar.Core` — memory plumbing + the PoE2 offset table + the live read layer. Read-side.
- `src/POE2Radar.Overlay` — tick loop, Direct2D overlay, HTTP API, read-only controller input. The deliverable `.exe`.
- `src/POE2Radar.Research` — dev-time discovery/validation tooling. Never linked into the overlay.

## Architecture

**Entry point:** `src/POE2Radar.Overlay/Program.cs` — attach (`ProcessHandle.AttachToPoE`) →
`Bootstrap.ResolveGameStateSlot` (AOB scan for the GameState pointer, validated by a working chain)
→ `RadarApp.Run`.

**Core read layer:**
- `MemoryReader.cs`, `ProcessHandle.cs`, `Native/` — Win32 + typed reads. `AttachToPoE` lists the
  PoE2 client process names.
- `Game/Poe2Offsets.cs` — **single source of truth for all PoE2 offsets** (validated + GameHelper2-
  sourced; markers `✓` = confirmed live).
- `Game/Poe2Live.cs` — the live reader: resolves GameState → InGameState → AreaInstance →
  LocalPlayer each tick; reads player vitals, walks the entity std::maps into categorized dots
  (rarity, reaction/hostility, POI via MinimapIcon, HP), reads the walkable terrain grid, the map
  UI element (visibility/shift/zoom), tile landmarks, and area/character info. Caches per-entity
  component addresses; cache key is the AreaInstance address (invalidates on zone change).
- `Game/GameStructs.cs` — blittable structs (`StdVector`, `Vector2/3`, `VitalStruct`).
- `Game/AobScanner.cs` + `AobPatterns.cs` — pattern scan for the GameState global slot.
- `Game/LifeValidator.cs` — value-scan to find the Life component by HP (Research `--hp`).
- `Game/ItemModTranslator.cs` — renders item mods (internal mod id + rolled values read from memory) to
  the game's English stat lines (e.g. `IncreasedLife5 [67]` → "+67 to maximum Life"). Two embedded RePoE
  PoE2 tables (`poe2_mod_stats.json` mod→stat-ids, `poe2_stat_descriptions.json` GGG stat descriptions,
  `poe2_mod_ranges.json` per-mod roll ranges), regenerated per patch via the `POE2Radar.Research --gen-*`
  probes (e.g. `--gen-weights`, `--gen-ranges`) from vendored RePoE snapshots under `resources/poe2-data/`
  (there is no `regenerate.py`). Validate with `--inventory --itemmods`.
- `Pathfinding/MapProjection.cs` + `GridConstants.cs` — isometric grid→screen projection and the
  grid↔world scale (250/23 ≈ 10.87).

**Overlay** (`src/POE2Radar.Overlay/`):
- `RadarApp.cs` — **two threads** (the render thread is never blocked by the heavy walk):
  - *Render loop* (`Tick`, ~`FpsCap` Hz): fast per-frame reads on its OWN reader stack (`_liveRender`) —
    live player/vitals/camera/map + auto-flask + HP-bar live pos — then draws from the lock-free
    published snapshots. Publishes `RadarState` for the API.
  - *World loop* (`WorldLoop`→`WorldTick`, ~30 Hz, own thread + reader `_live`): the heavy entity/terrain/
    landmark walk + mod catalog + HP-bar specs + item labels + atlas update + nav/route maintenance.
    Publishes an immutable `WorldSnapshot` (+ a separate `AtlasRender` bundle) the render thread reads
    lock-free (volatile reference swap, same idiom as `_state`).
  - Three INDEPENDENT reader stacks over the one `ProcessHandle` (RPM is concurrency-safe; the per-instance
    buffers/caches are NOT): `_live` (world), `_liveRender` (render), `_liveApi` (HTTP/tile scans). `_atlas`
    is internally locked, so it's shared. HP-bar live reads carry the mob's Render/Life component addresses
    in the spec (`Poe2Live.TryBarComponents`/`TryLiveBarAt`) so the render thread reads them with no shared

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luther-rotmg/POE2GPS](https://github.com/luther-rotmg/POE2GPS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
