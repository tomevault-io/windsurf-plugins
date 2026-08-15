---
trigger: always_on
description: 이 문서는 Flip Friends 저장소에서 작업할 때 따를 가이드라인입니다. 모든 소스 파일은 **UTF-8** 인코딩으로 저장합니다.
---

# Repository Guidelines

이 문서는 Flip Friends 저장소에서 작업할 때 따를 가이드라인입니다. 모든 소스 파일은 **UTF-8** 인코딩으로 저장합니다.

## Documentation Entry Point

작업을 시작하기 전에 [`PROJECT_DOCUMENTATION.md`](PROJECT_DOCUMENTATION.md)의 작업별 문서 경로를 확인하고, 해당 기능 문서만 추가로 읽습니다. 프로젝트의 현재 진행 상태와 다음 작업은 이 파일에 중복 기록하지 않습니다.

## Project Overview

**Flip Friends** is a 2D multiplayer co-op platformer built with Unity and Mirror Networking. Up to 4 players work together to climb levels, solve puzzles, carry objects/players, and reach finish points. Uses Steam (FizzySteamworks) for lobby management.

## Project Structure & Module Organization

This repository is a Unity 6 project (`ProjectSettings/ProjectVersion.txt` shows `6000.5.0f1`). Game code lives primarily in `Assets/01_Scripts`, organized by feature such as `NetworkScripts`, `GameObjScripts`, `GameOption`, and `UI Scripts`. Reusable content is under `Assets/03_Prefabs`, sprites under `Assets/02_Sprites`, and shared fonts/settings at the `Assets` root. Package configuration is in `Packages/manifest.json`; project-wide editor and build settings live in `ProjectSettings/`.

Third-party code is checked into `Assets`, especially `Assets/Mirror`, `Assets/Plugins/Demigiant`, and `Assets/com.rlabrecque.steamworks.net`. Treat those as vendor code unless a task explicitly requires patching them.

## Architecture

### Networking Model (Mirror Framework)

- **Server Authority**: All gameplay logic (movement, physics, collision) runs on server — `MovementHandler.FixedUpdate` has a hard `if (!isServer) return` guard
- **Client-to-Server**: Use `[Command]` for player actions (e.g., `CmdJumpInputDown`, `CmdObjectInteraction`)
- **Server-to-Clients**: Use `[ClientRpc]` for state broadcasts (e.g., `RpcFlipChanged`, `RpcVelocityReset`)
- **State Sync**: Use `[SyncVar(hook = nameof(...))]` for automatic hook callbacks on all clients
- Always check `isServer` / `isOwned` / `isLocalPlayer` before executing context-specific code

### Core Script Architecture

**Network Layer** (`Assets/01_Scripts/NetworkScripts/`):

- `SteamRoomManager.cs` — Steam lobby creation, join codes, Steamworks callbacks; holds `playerName`
- `SlimeRoomManager.cs` — Extends `NetworkRoomManager`; holds `currentStage` (int index into `StageManager.stageMapPrefabs`); handles scene transitions and player reconnection after stage clear
- `CustomRoomPlayer.cs` — Lobby player state (name, color, ready state via `SyncVar`); bridges to `PlayerController2D` after scene load

**Player System** (`Assets/01_Scripts/PlayerScripts/New Scripts/`):

- `PlayerController2D.cs` — Central coordinator; owns all subsystems, handles `[Command]` dispatch, finish state, and damage routing. `Update` (client-side input) calls Commands; `FixedUpdate` (server-side) drives state/animation
- `PlayerInputManager.cs` — New Input System callbacks; exposes input state as properties (`IsJumpPressed`, `MovementInput`, etc.)
- `MovementHandler.cs` — Physics-based movement (server-only `FixedUpdate`); gravity derived from `maxJumpHeight`/`timeToJumpApex`; handles wall jumping, rope climbing, conveyor acceleration, invincibility, and knockback
- `Controller2D.cs` — Raycast collision system; exposes `collisions` struct (`below`, `above`, `left`, `right`, `slidingDownMaxSlope`, `slopeNormal`); tracks `underPlayer` and `onConveyor`
- `RaycastController.cs` — Base for `Controller2D` and `Switch`; manages ray origins and spacing
- `PlayerInteraction.cs` — Pickup/carry/throw for both `PickupObj` and other `PlayerController2D` instances; uses multi-ray box scan on "Pickable" and "Player" layers

**Game Objects** (`Assets/01_Scripts/GameObjScripts/` & `ObstacleScripts/`):

- `PickupObj.cs` — Carryable objects with physics and network sync
- `MovingPlatform.cs` — Lerp-based platforms with RPC position sync
- `BasicTrap.cs` — Base trap class; exposes `knockbackDir` for custom knockback direction
- `RotatingObstacle.cs` — Extends `BasicTrap`
- `Conveyor.cs` — Directional movement surface; `isClockwise` controls direction; integrated via `Controller2D.onConveyor`

**Puzzle System** (`Assets/01_Scripts/PlayingScripts/SwitchScripts/`):

- `Switch.cs` — Abstract base extending `RaycastController`; `SyncVar isActivated`; `DetectPlayer` uses downward raycasts; subclasses override `OnSwitchStateChanged`
- `LayerBasedSwitch.cs` — Trigger-based activation
- `DoorSwitch.cs` — Controls door state via RPC

**Game Management**:

- `GameManager.cs` — Singleton; `FinishCheck()` called by `PlayerController2D` when `isFinish` SyncVar changes; triggers `SlimeRoomManager.ReturnRoomScene()` when all players finish
- `StageManager.cs` — Server-only; reads `SlimeRoomManager.currentStage` in `OnStartServer`, instantiates and `NetworkServer.Spawn`s the stage prefab
- `RespawnHandler.cs` — Layer-based respawn triggers; `SavePoint.cs` tracks ordered save points by `savePointID`

### Player States

`PlayerState` enum (in `PlayerController2D.cs`): `Idle`, `Walk`, `Jump`, `Damaged`, `Attack`, `Climb`, `ClimbIdle`, `Shrink`, `Carried`, `Throw`

### Input System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jsg9147/Flip-Friends](https://github.com/jsg9147/Flip-Friends) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
