---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

GeminiStarPlatinum is an Unreal Engine 5.6 real-time **physics simulation of the Gemini telescope and its observatory dome**. Movements are driven by physics constraints (not animation), and the longer-term intent is to mirror a live TCS (Telescope Control System) EPICS data feed. Single C++ Runtime module: `GeminiStarPlatinum`.

## Project Goals

These are the guiding objectives for the project. Weigh new work against them.

- **Immersive operator experience** — Recreate a simplified version of what telescope operators do in the command center. Users should be able to observe and interact with the live behavior of the telescope and dome, with access to the important positional data points (e.g. azimuth/elevation/Cassegrain angles, dome twist/shutter/vent state) as they move.
- **Streamlined, dual-audience data visualization** — Build a data interface that serves both laymen and engineers/operators. Support both at-a-glance world-space/camera-space figures (labels and readouts attached to the moving geometry) and detailed drill-down menus for precise numeric state. Keep the default view approachable while making the deeper data available on demand.
- **Maintainability and extensibility** — Contribute so future work doesn't require extensive re-learning. Favor clear MVC boundaries (models as source of truth, actors as views, coordinator/feed as the control source), document non-obvious physics and wiring decisions inline, and design new UI/data features so additional data points or panels can be added without rework.

## Build & Run

Workflow is **Editor + Live Coding** with a default Epic Games Launcher engine install (`C:\Program Files\Epic Games\UE_5.6`).

- **Run / open project**: open `GeminiStarPlatinum.uproject` (double-click or via Epic Launcher). If C++ binaries are stale it prompts to rebuild on launch. Default startup map is `/Game/MainWorld`.
- **Rebuild C++ while the editor is open**: **Live Coding** — `Ctrl+Alt+F11`. Prefer this for iterating on `.cpp` changes. Header/`UCLASS`/`UPROPERTY` signature changes generally require a full editor restart + rebuild.
- **Regenerate Visual Studio project files** (after adding/removing source files): right-click `GeminiStarPlatinum.uproject` → "Generate Visual Studio project files".
- **Full CLI build** (editor closed):
  ```
  "C:\Program Files\Epic Games\UE_5.6\Engine\Build\BatchFiles\Build.bat" GeminiStarPlatinumEditor Win64 Development -Project="<abs path>\GeminiStarPlatinum.uproject" -WaitMutex
  ```

There is **no automated test suite** (no Automation specs, no test target). Don't go looking for one; verify changes by running the simulation in the editor.

## Coding/Documentation style

- When commenting functions, use Doxygen-style XML comments for documentation generation purposes.
- Be highly concise and precise when describing functions and variables. Include relevant units and ranges (may point to relevant DataAsset instead if it exists)

## Git 

Do NOT stage or push commits without the express permission of the user

## Architecture — MVC (migration in progress)

The codebase is mid-refactor toward an MVC split (noted explicitly in `AssemblyModel.h`). Understanding it requires reading the model subsystems and the actors together:

- **Models** — `UAssemblyModel` (abstract) → `UTelescopeModel`, `UDomeModel`. These are `UGameInstanceSubsystem` singletons that hold target state (e.g. `AzimTarget`/`ElevTarget`/`CassTarget`, dome twist/shutter/vent targets), clamp inputs to per-axis limits, and broadcast `FOnStateChanged`. They are the source of truth.
- **Actors / Views** — `AMovingThing` (base) → `AMovingTelescope`, `AMovingDome`. Physics actors that build their component + constraint hierarchy in the constructor and, each `Tick`, read the model's targets and drive `UPhysicsConstraintComponent`s toward them. Models are fetched lazily, e.g. `GetGameInstance()->GetSubsystem<UTelescopeModel>()`.
- **Coordinator / Feed** — `UObservatoryCoordinator` (a `UGameInstanceSubsystem` holding `EControlMode { Manual, Live }`) owns a single `ULiveDataFeed` and brokers the control source: `SetControlMode(Live)` connects the feed, `Manual` disconnects it, and it broadcasts `OnControlModeChanged`. `ULiveDataFeed` is an implemented `FTickableGameObject` TCP/JSON-lines **client** that streams positional samples from an external bridge and pushes them into the model setters. Both are wired up — not stubs. See **Live data feed & TestIOC** below.

**Wiring status (important):** both actors are now wired to their models. The telescope reads `AzimTarget`/`ElevTarget`/`CassTarget` from `UTelescopeModel`; the dome reads `DomeTwistTarget`/`TopSSwingTarget`/`BotSSwingTarget`/`VentSlideTarget` from `UDomeModel`. The open/closed target values live only in `UDomeModel::SetOpen` (the previously duplicated block in `AMovingDome::Tick` has been removed), so the model is the single source of truth.

## Live data feed & TestIOC


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gemini-rtsw/GeminiStarPlatinum](https://github.com/gemini-rtsw/GeminiStarPlatinum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
