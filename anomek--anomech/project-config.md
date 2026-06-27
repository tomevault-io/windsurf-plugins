---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

AnoMech ("Another FFXIV mechanics simulator") is a Dalamud plugin that **simulates FFXIV raid mechanics client-side** — it spawns fake `BattleChara` instances (party doppels and bosses) into the live game, drives their actions, and renders the canonical VFX/cast bars/tethers so players can practice mechanics solo. Currently focused on ultimate-raid phases (TOP P5 Delta / Sigma) but the engine is general-purpose. It is NOT the SamplePlugin template the README still describes; only the project skeleton was inherited.

The reference scenario is **TOP P5 Delta** (`Scenarios/Top/P5Delta/`). Treat it as the canonical example of how a scenario consumes the engine — when designing new APIs, look at how it would read there. Scenarios are nested by family (e.g. `Scenarios/Top/`); IDs that recur across phases of the same family live in a shared `<Family>Constants.cs` (e.g. `Scenarios/Top/TopConstants.cs`) and are referenced from phase scenarios via the fully-qualified path `TopConstants.<Group>.<Const>`.

## Build / run

- Build: `dotnet build` from `D:/Projects/ffxiv/AnoMech/`. Output is `bin/Debug/AnoMech.dll`.
- The csproj uses `Dalamud.NET.Sdk/15.0.0` — Dalamud SDK resolves at build time from `%AppData%/XIVLauncher/addon/Hooks/dev/`. No NuGet restore tweaks are needed.
- **There are no automated tests.** Verification is "build clean → load DLL via Dalamud Dev Plugins → run a scenario in-game and watch." For UI / behavior changes, ask the user to run the plugin; you cannot.
- In-game entry point: chat command `/anomech` (alias `/ano`) opens the main window. Subcommands: `config`, `start`, `reset`, `leave`. Buttons in the main window run scenarios and despawn/reset.

## Architecture

### Frame loop
`Plugin.OnFrameworkUpdate` → `Game.Tick(dt)` → `SimWorld.Tick(dt)` → ticks `EventScheduler` (scaled by `EventTimeScale`), then each `SimEnemy`, `SimParty` (which ticks each `SimPartyMember`), `LocalPlayerOps`, each `SimTether`, then refreshes `EnmityHud` and `PartyHud`. Everything runs on the Framework thread.

### Two-layer Core

**`Core/SimObjects/`** — in-world entities. All implement `ISimObject` (`bool IsAlive`, `Tick(float)`, `Despawn()`); position-bearing ones implement `IPositioned`. The contract and design rules live in the header comment of `ISimObject.cs` — read it before adding a new SimObject type. Core types: `SimWorld` (root), `SimNpc` (base wrapper around a `BattleChara*`), `SimEnemy : SimNpc` (cast bars, action timeline), `SimPartyMember : SimNpc` (CharacterManager registration), `SimParty` (8-slot container), `SimTether` (effect with auto-expire). **Spawn through the parent** (`SimWorld.SpawnEnemy`, `SimWorld.InitializeParty`, `SimWorld.Tether`) — never `new` these from outside.

**`Core/` (root)** — helpers. Engine plumbing that is *not* a game object: `VfxFunctions` (signature-scanned native VFX/tether functions), `Statuses` (direct `StatusManager` writes that bypass server packets), `PinnedStatus` / `TimedStatus` (status-lifetime helpers re-stamped each tick), `LocalPlayerOps` (VFX/status on the real local player), `PartyHud` / `EnmityHud` (mirror SimObjects state into game UI addons via `AddonLifecycle.PreRequestedUpdate`), `EventScheduler`, `TetherTarget` (unifies `SimNpc` and the local player), `PartyMemberOrPlayer` (lets scenarios address all 8 slots uniformly with player-fallback), `MathUtil`, `PartyPresets`, `Waymarks`. These do NOT go in SimObjects.

### Scenarios
`Scenarios/IScenario.cs` is the contract: `Run(SimWorld, PartyRole)`. `Game.RunScenario` wires up origin snapshot, party init, waymarks, then invokes `Run`. A scenario typically declares its event timeline via `world.Events.Add(offset, action)` — the EventScheduler then fires each lambda at the scheduled time. The TOP P5 Delta scenario at `Scenarios/Top/P5Delta/` is split into `TopP5DeltaScenario` (event timeline + spawns + casts), `TopP5DeltaAi` (party-member movement choreography), `TopP5DeltaState` (per-run randomization), `TopP5DeltaConstants` (phase-specific BNpc / action / status / tether IDs), `TopP5DeltaSettingsWindow` + `TopP5DeltaStateOverrides` (debug overrides). Family-shared IDs (boss BNpcBase rows, Hello-World action IDs, arena radius) live in `Scenarios/Top/TopConstants.cs`.

### Scenario timeline conventions
- **Use absolute time literals in `world.Events.Add`.** Every entry in a scenario's `Run` should be `world.Events.Add(<absolute t>, ...)` from scenario start (e.g. `55f`, `56.5f`) — not `<base> + offset` arithmetic, named time constants, or chained `Events.Add` calls inside event handlers. The whole scenario timeline should be readable top-to-bottom as a single list of absolute timestamps. If state needs to flow between events, store it on `TopP5DeltaState` (or the equivalent scenario-state object) and have each handler read/mutate it.

### Heavy native interop

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anomek/AnoMech](https://github.com/anomek/AnoMech) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
