---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**TeaLeaves** is a Godot 4.6 project using **C# for all gameplay logic** with GDScript reserved for editor tooling only. It uses Jolt Physics and Forward Plus rendering with D3D12 on Windows.

## First-Time Setup

Before working on this project for the first time:

```powershell
# 1. Restore NuGet packages (required for C# compilation)
dotnet restore

# 2. Verify C# builds successfully
dotnet build -warnaserror

# 3. Initialize input actions (adds WASD, jump, etc. to project.godot)
pwsh ./tools/godot.ps1 --headless --script res://tools/setup_input_actions_cli.gd

# 4. Run tests to verify everything works
dotnet test
pwsh ./tools/test.ps1
```

If `dotnet build` fails with missing SDK errors, ensure you have .NET 8.0 SDK installed and Godot 4.6+ Mono.

## Core Tenets

- **C# for gameplay**, GDScript only for editor tooling or tiny glue
- **NEVER write GDScript** for gameplay unless absolutely necessary
- **Composition over inheritance** for Nodes
- **Typed EventBus** for cross-system communication
- **Data-driven configs** using Godot Resource assets (`[GlobalClass]` C# classes)
- **Fail-fast validation**: Misconfigured objects are disabled and logged via `GD.PushError()`
- **Deterministic state machines** with explicit state transitions
- **Test-driven debugging**: Create a failing test first, verify it fails, fix the code, verify the test passes

## Language Usage

- **C#** for all gameplay logic, systems, and tests
- **GDScript** ONLY for editor tools and scene glue scripts
- Never use GDScript for gameplay logic

### Namespace Convention
```
TeaLeaves.*           # All gameplay code
TeaLeaves.Systems.*   # Core systems (EventBus, state machines, etc.)
```

## EventBus Pattern

The EventBus provides **typed, decoupled communication** between systems. Use it when:
- Systems need to communicate without direct references
- Multiple listeners need to respond to the same event
- You want to avoid tight coupling between game systems

### When to Use EventBus vs Godot Signals

| Use EventBus | Use Godot Signals |
|--------------|-------------------|
| Cross-system communication (UI ↔ Gameplay) | Parent-child node communication |
| Global events (game state changes, achievements) | Local component events (animation finished) |
| Multiple unrelated listeners | Single known listener |
| Events that need to survive scene changes | Scene-local events |

### EventBus Implementation

Create the EventBus as an AutoLoad in `game/EventBus.cs`:

```csharp
using Godot;
using System;
using System.Collections.Generic;

namespace TeaLeaves.Systems
{
    /// <summary>
    /// Typed event bus for decoupled cross-system communication.
    /// Register as AutoLoad: EventBus="*res://game/EventBus.cs"
    /// </summary>
    public partial class EventBus : Node
    {
        public static EventBus Instance { get; private set; }

        public override void _Ready()
        {
            Instance = this;
        }

        // --- Event Delegates (define one per event type) ---

        public delegate void PlayerDamagedHandler(int damage, Node3D source);
        public delegate void ItemCollectedHandler(string itemId, int quantity);
        public delegate void GameStateChangedHandler(GameState oldState, GameState newState);
        public delegate void InteractionHandler(Node3D target, string verbId);

        // --- Events (subscribers connect to these) ---

        public event PlayerDamagedHandler PlayerDamaged;
        public event ItemCollectedHandler ItemCollected;
        public event GameStateChangedHandler GameStateChanged;
        public event InteractionHandler InteractionStarted;
        public event InteractionHandler InteractionCompleted;

        // --- Emit Methods (publishers call these) ---

        public void EmitPlayerDamaged(int damage, Node3D source)
        {
            PlayerDamaged?.Invoke(damage, source);
        }

        public void EmitItemCollected(string itemId, int quantity = 1)
        {
            ItemCollected?.Invoke(itemId, quantity);
        }

        public void EmitGameStateChanged(GameState oldState, GameState newState)
        {
            GameStateChanged?.Invoke(oldState, newState);
        }

        public void EmitInteractionStarted(Node3D target, string verbId)
        {
            InteractionStarted?.Invoke(target, verbId);
        }

        public void EmitInteractionCompleted(Node3D target, string verbId)
        {
            InteractionCompleted?.Invoke(target, verbId);
        }
    }

    // Define enums/types used by events
    public enum GameState { MainMenu, Playing, Paused, Cutscene, GameOver }
}
```

### Subscribing to Events

```csharp
using Godot;
using TeaLeaves.Systems;

public partial class HealthBar : Control
{
    public override void _Ready()
    {
        // Subscribe to event
        EventBus.Instance.PlayerDamaged += OnPlayerDamaged;
    }

    public override void _ExitTree()
    {
        // CRITICAL: Always unsubscribe to prevent memory leaks
        EventBus.Instance.PlayerDamaged -= OnPlayerDamaged;
    }

    private void OnPlayerDamaged(int damage, Node3D source)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cleak/quasar-saz](https://github.com/cleak/quasar-saz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
