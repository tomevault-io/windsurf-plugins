---
trigger: always_on
description: MixScrims is a **SwiftlyS2 plugin** that implements FACEIT-style PUG matches with in-game management. It's a **state machine-driven plugin** that progresses through match phases: Warmup → MapVoting → MapChosen → PickingTeam → KnifeRound → PickingStartingSide → Match.
---

# MixScrims CS2 Plugin - AI Coding Agent Instructions

## Project Overview

MixScrims is a **SwiftlyS2 plugin** that implements FACEIT-style PUG matches with in-game management. It's a **state machine-driven plugin** that progresses through match phases: Warmup → MapVoting → MapChosen → PickingTeam → KnifeRound → PickingStartingSide → Match.

- **Plugin Framework**: [SwiftlyS2](https://swiftlys2.net) - CS2 server modification framework for .NET 10.0
- **Plugin Version**: 1.6.1
- **Architecture**: State-based partial classes with shared service layer
- **Key Components**: Main plugin (`MixScrims`), Contract API (`MixScrims.Contract`), state handlers, shared services, announcement system

**Documentation:**
- [Project Wiki](https://github.com/shmitzas/MixScrims-SwiftlyS2/wiki) - Comprehensive guides for installation, configuration, features, and contributing
- [SwiftlyS2 API Reference](https://swiftlys2.net/llms-full.txt) - Framework API documentation

**Jump to:** [Quick Reference](#quick-reference) • [Match Flow](#match-flow-overview) • [Architecture](#critical-architecture-patterns) • [Development](#development-workflows) • [Config](#config-features-by-category) • [API](#shared-api-pattern) • [Commands](#available-commands) • [Testing](#testing--debugging) • [Pitfalls](#common-pitfalls)

## Quick Reference

**When you need to:**
- **Modify state logic**: Check `src/States/{StateName}/` (Main.cs or Events.cs)
- **Add cross-state features**: Edit `src/StateAgnostic/Main.cs` or `Announcements.cs`
- **Add new commands**: Update `src/Commands/` + `Config.Commands` dictionary
- **Extend public API**: Add methods to `IMixScrims.cs` + implement in `MixScrimsService.cs`
- **Change announcements**: Modify `resources/translations/{lang}.jsonc`
- **Adjust match flow**: Edit CFG files in `csgo_configs/` + config.jsonc

**Critical Rules:**
- Always use `mixScrimsService.SetMatchState()` - never modify state directly
- Check player validity: `if (!player.IsValid || !player.IsConnected || player.IsBot)`
- Store scheduler tokens: `Core.Scheduler.StopOnMapChange(token)`

## Match Flow Overview

**Full Sequence:** Warmup → Map Voting → Map Chosen → Team Picking → Knife Round → Side Selection → Match → Ended

**Key Transitions:**
- **Warmup**: Players `!ready` until `MinimumReadyPlayers` met → advances to Map Voting (or Map Chosen if `SkipMapVoting`)
- **Map Voting**: 30s vote window, highest votes wins → map change → Map Chosen state
- **Map Chosen**: Players `!ready` again after map load → Captains selected (random/volunteer/manual) → Team Picking
- **Team Picking**: Captains alternate picking players via menu (random captain picks first) → Knife Round when complete
- **Knife Round**: Knife-only round, winning team captain gets menu → Side Selection
- **Side Selection**: Winner chooses `!stay` or `!switch` sides → Match starts with `match_start.cfg`
- **Match**: Standard MR12 competitive, team timeouts (vote-based, 3 per team), surrender voting → Surrender (if team votes) OR Ended → resets to Warmup

**Config Shortcuts:** `SkipMapVoting` (uses current map), `SkipTeamPicking` (random teams), `DisableCaptains` (team voting for side selection)

## Critical Architecture Patterns

### Partial Class Structure
The main `MixScrims` class is split across **multiple partial class files** organized by concern:
- `src/Main.cs` - Core plugin lifecycle, DI setup, command registration
- `src/StateAgnostic/Main.cs` - Cross-state logic (ready checks, team management, player punishment)
- `src/StateAgnostic/Announcements.cs` - Ready status announcements, scoreboard/center HTML display
- `src/StateAgnostic/Events.cs` - Cross-state event handlers
- `src/States/{StateName}/Main.cs` - State-specific logic (when needed)
- `src/States/{StateName}/Events.cs` - State-specific event handlers

**State Folder Structure:**

| State | Main.cs | Events.cs | Extra Files | Purpose |
|-------|---------|-----------|-------------|---------|  
| KnifeRound | ✅ | ✅ | — | Knife round logic & round end handling |
| MapChosen | ❌ | ✅ | — | Post-map-vote ready check |
| MapVoting | ✅ | ❌ | — | Vote collection & map selection |
| Match | ✅ | ✅ | `VoteKick.cs` | Active match logic, round events & votekick |
| PickingTeam | ✅ | ❌ | — | Captain team selection menus |
| Reset | ✅ | ❌ | — | Plugin state reset & cleanup |
| Surrender | ✅ | ❌ | — | Team surrender voting |
| Timeout | ✅ | ✅ | — | Pause/resume logic & timer |
| VoteKick | ❌ | ❌ | — | Reserved folder (logic lives in Match/VoteKick.cs) |
| Warmup | ✅ | ✅ | — | Initial ready check & player join handling |

When modifying functionality, **always check if related code exists in other partial files** before adding new methods.

### State Machine Management
State transitions are managed through `MixScrimsService.SetMatchState(MatchState state)`:
```csharp
// States defined in MixScrims.Contract/MatchState.cs
mixScrimsService.SetMatchState(MatchState.MapVoting);
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shmitzas/MixScrims-SwiftlyS2](https://github.com/shmitzas/MixScrims-SwiftlyS2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
