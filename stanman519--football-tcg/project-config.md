---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**First & Long** — a football-themed trading card game built on **Unity 6** (URP) using the **TcgEngine** template. C# only. Networking via Unity Netcode for GameObjects.

## Build & Run

This is a Unity project — there is no CLI build. Open in Unity Editor (6.x), hit Play.

- **Solution**: `My project.sln` / `Assembly-CSharp.csproj`
- **No test suite** — no unit tests exist. Test by running in Unity Editor.
- **Card import**: Unity menu `First&Long > Import Player Cards from CSV` (reads `Assets/Resources/First and Long - card_exporter.csv`)
- **Play enhancer builder**: Unity menu `First&Long > Create Play Enhancer Cards`

## Architecture

### Three Pillars

| Class | Namespace | Role |
|---|---|---|
| `GameLogicService` | `Assets.TcgEngine.Scripts.Gameplay` | Core rules engine (~2700 lines). All game rules, phase transitions, ability resolution, yardage calc |
| `GameServer` | `TcgEngine.Server` | Server-side. Receives actions, validates, runs AI. Owns `GameLogicService` |
| `GameClient` | `TcgEngine.Client` | Client singleton MonoBehaviour. Sends actions, receives state refreshes |

### Data Flow

```
Player Input -> GameClient.SendAction() -> Network -> GameServer.ReceiveAction()
    -> GameLogicService (mutates Game state) -> onRefresh Event -> Client UI update
```

### Namespace Mess (Important)

Three namespaces coexist — watch imports when creating new files:
- `TcgEngine` — data classes (CardData, AbilityData, enums), base EffectData/ConditionData, most template conditions
- `Assets.TcgEngine.Scripts.Gameplay` — runtime gameplay (Game, Player, Card, GameLogicService, HeadCoachCard)
- `Assets.TcgEngine.Scripts.Effects` — custom effects
- `Assets.TcgEngine.Scripts.Conditions` — some custom conditions (FirstPlay, PlayType, FirstSnap)
- `TcgEngine.Conditions` — other custom conditions (PlaysRemaining, DownAndDistance, GritCompare)
- `TcgEngine.Client` — all GameClient-side scripts
- `TcgEngine.Server` — GameServer
- `TcgEngine.AI` — AI players

### Ability System (Trigger -> Condition -> Effect)

Cards have `AbilityData[]` abilities. Each ability has:
- **Trigger** (`AbilityTrigger` enum): When it fires (OnPlay, OnRun, OnPass, StartOfTurn, OnLiveBallResolution=55, etc.)
- **Conditions** (`ConditionData[]`): ScriptableObject subclasses, checked via `IsTriggerConditionMet()`/`IsTargetConditionMet()`
- **Effects** (`EffectData[]`): ScriptableObject subclasses, executed via `DoEffect()` overloads
- **SlotRequirements** (`SlotRequirement[]`): slot icons needed to activate

New conditions/effects: subclass `ConditionData`/`EffectData`, create as ScriptableObject asset in `Assets/Resources/Conditions/` or `Assets/Resources/Effects/`. Register via Unity's `[CreateAssetMenu]`.

### Game Phase Loop

```
StartTurn -> ChoosePlayers -> RevealPlayers -> ChoosePlay -> RevealPlayCalls
    -> SlotSpin -> ResolvePlayOutcome -> [LiveBall if ball live] -> EndTurn
```

Phase enforcement is in `Game.CanPlayCard()`. Card types are phase-locked:
- `OffensivePlayer`/`DefensivePlayer` -> `ChoosePlayers` only
- `OffensivePlayEnhancer`/`DefensivePlayEnhancer` -> `ChoosePlay` only (max 1 per player)
- `OffLiveBall`/`DefLiveBall` -> `LiveBall` only

### Card Data

`CardData` (ScriptableObject) in `Assets/Resources/Cards/*.asset`. Key stats: `stamina`, `grit`, `run_bonus`, `short_pass_bonus`, `deep_pass_bonus`, `run_coverage_bonus`, `short_pass_coverage_bonus`, `deep_pass_coverage_bonus`, `isSuperstar`, `playerPosition`, `suit`, `slotRequirements[]`.

Runtime card state is `Card` class (in `Card.cs`), wraps `CardData` with mutable fields.

### Yardage Calculation

**Run**: `(coachBase + playerRunBase + statusBonuses) - (defCoverageBase + defBonuses)`
**Pass**: `topReceiverBase + topReceiverStatusBonus + otherOffBonuses + coachBase - defYardage`

Pass completion check reads middle reel icon from each of the 3 slot reels.

### Suit System

One card per suit per turn (player cards). `CardSuit` enum: None/Clubs/Diamonds/Hearts/Spades. Tracked in `player.suits_played_this_turn`, enforced in `CanPlayCard`.

### Slot Machine

3 reels, 8 symbols each, 512 middle-row combos. Wrench = outer reels only (bad events). Wild = center reel only.

### CSV Importer Column Layout

0=card_id, 1=Name, 2=pos, 3=Superstar?, 4=passive(text), 5=Suit, 6-8=RunBonus/ShortPass/DeepPass, 9-11=RunCov/ShortCov/DeepCov, 12=Stamina, 13=Grit. Ability slots start at col 14 (9 cols each, up to 2 slots).

## Key Files

- **Rules engine**: `Assets/TcgEngine/Scripts/Gameplay/GameLogicService.cs`
- **Game state**: `Assets/TcgEngine/Scripts/Gameplay/Game.cs`
- **Player state**: `Assets/TcgEngine/Scripts/Gameplay/Player.cs`
- **Card runtime**: `Assets/TcgEngine/Scripts/Gameplay/Card.cs`
- **Card data def**: `Assets/TcgEngine/Scripts/Data/CardData.cs`
- **Ability data def**: `Assets/TcgEngine/Scripts/Data/AbilityData.cs`
- **Slot machine**: `Assets/TcgEngine/Scripts/Gameplay/SlotMachineManager.cs`
- **Receiver ranking**: `Assets/TcgEngine/Scripts/Gameplay/ReceiverRankingSystem.cs`
- **CSV importer**: `Assets/TcgEngine/Editor/PlayerCardImporter.cs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Stanman519) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
