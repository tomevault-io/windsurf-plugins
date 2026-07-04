---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

Last Updated: 2026-03-10

---

# Build & Run Commands

## Server
```bash
# Build server solution
dotnet build Server/Server.sln

# Run server
dotnet run --project Server/Main/Main.csproj

# Or open Server/Server.sln in Visual Studio/Rider and run Main project
```
Server listens on `127.0.0.1:20000` (KCP protocol) by default.

## Client
```bash
# Open in Unity Editor (Unity 2022.3.62f2 required)
# Open Client/Unity/Unity.sln for code editing
# Or open Client/Unity project directly in Unity Editor
```

## Config Generation (Luban)
```bash
# Generate config tables from Excel to Binary
# Config source: Config/Excel/
# Config output: Config/Binary/
# (Tool: Luban)
```

---

# Project Overview

This repository contains a **technical demo project** integrating:

- Unity client using **TEngine**
- .NET server using **Fantasy**
- Deterministic frame synchronization gameplay

The goal of the project is to learn and demonstrate:

- Client/server architecture
- Frame synchronization networking
- Deterministic simulation
- Fighting game style gameplay

The project is **NOT a production game**, but a learning and architecture demo.

---

# Technology Stack

Client

- Unity 2022.3.62f2
- TEngine framework
- HybridCLR (HotFix assembly)
- YooAsset (resource management)
- UniTask (async)

Server

- .NET 9
- Fantasy framework
- KCP networking
- Entitas ECS components

Tools

- Luban (Excel → config tables)
- Odin Inspector

---

# Repository Structure

## Client

Client/Unity

Important directories:

- Assets/GameScripts
- Editor
- Main
- HotFix

HotFix contains most gameplay logic.

Assets/GameScripts/HotFix

Important submodules:

- Procedure
- Net
- Battle
- Deterministic
- View

---

## Server

Server

Projects:

- Main
- Entity
- Hotfix

Responsibilities:

- **Main**: Server entry point
- **Entity**: Shared data structures and deterministic simulation
- **Hotfix**: Gameplay logic systems

---

# Deterministic Simulation (VERY IMPORTANT)

This project uses **deterministic simulation** for frame synchronization.

**ALL GAMEPLAY LOGIC MUST USE FIXED POINT MATH.**

Floating point numbers MUST NOT be used in gameplay simulation.

Do NOT use:
- `float`
- `double`
- `UnityEngine.Vector3`
- `UnityEngine.Time.deltaTime`

Instead use the deterministic math library.

---

# Deterministic Library Location

Client: `Client/Unity/Assets/GameScripts/HotFix/GameLogic/Deterministic/`

Server: `Server/Entity/Deterministic/`

The client and server must share **identical deterministic math implementations**.

The deterministic layer includes:

- `Fix64` - 64-bit fixed-point number (Q31.32)
- `FixVector2` - 2D vector using Fix64
- `FixVector3` - 3D vector using Fix64
- `FixMath` - Math utility functions

These types should be used for:
- Position
- Velocity
- Collision
- Simulation logic

---

# Gameplay Architecture

The gameplay model is **server authoritative frame synchronization**.

Basic flow:

```
Client Input → Server collects inputs → Server broadcasts frame → Clients simulate frame
```

Target simulation rate: **30 FPS** (logic), **60 FPS** (rendering with interpolation)

---

# Network Flow

## Login

Client sends: `C2G_LoginRequest` (UserName, Password)

Server responds: `G2C_LoginResponse` (UnitId, ErrorCode)

After login the player enters Lobby.

---

## Battle (Local Single-Player Frame Sync)

Current implementation is **local single-player** frame synchronization.

Key components:
- `FrameRunner` - Core logic driver at 30 FPS, Singleton implementing IUpdate
- `InputCollector` - MonoBehaviour that collects Unity input and submits to InputBuffer
- `SimulationWorld` - ECS-style simulation with entities and systems
- `RenderInterpolator` - Interpolates 30 FPS logic to 60 FPS rendering
- `InputBuffer` - Dictionary-based frame input storage (120 frames history)

---

# Client Architecture

Client uses TEngine Procedure state machine.

Procedure flow:

```
LaunchProcedure → LobbyProcedure → BattleProcedure
```

## Key Components

**FantasyManager** (`GameLogic/NetWork/FantasyManager.cs`)
- Singleton MonoBehaviour for Fantasy framework integration
- Handles Scene, Session, and Unit management
- Login functionality

**LobbyProcedure** (`GameLogic/Procedure/LobbyProcedure.cs`)
- Initializes Fantasy framework
- Shows LobbyUI
- Listens for login completion event
- Transitions to BattleProcedure after successful login

**BattleProcedure** (`GameLogic/Procedure/BattleProcedure.cs`)
- Initializes FrameRunner (Singleton)
- Creates InputCollector (MonoBehaviour for input collection)
- Creates two Player entities with deterministic components
- Creates player view GameObjects (Cubes) and binds to RenderInterpolator
- Cleans up resources on exit

**FrameRunner** (`GameLogic/Battle/FrameSync/FrameRunner.cs`)
- Singleton implementing IUpdate interface
- Core 30 FPS logic driver
- Manages SimulationWorld, InputBuffer, RenderInterpolator
- Accumulates deltaTime and executes fixed logic frames
- Calculates interpolation alpha for rendering

**SimulationWorld** (`GameLogic/Battle/Simulation/SimulationWorld.cs`)
- ECS-style world with entities and systems
- Stores player entities and frame snapshots

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daydayasobi/LockstepArcade-TEngine-Fantasy-Demo](https://github.com/daydayasobi/LockstepArcade-TEngine-Fantasy-Demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
