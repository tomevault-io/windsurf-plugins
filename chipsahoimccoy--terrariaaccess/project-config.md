---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Terraria Access is a tModLoader mod that makes Terraria playable for blind and low-vision players. It provides speech narration for menus and in-game UI via multiple screen readers, plus positional audio cues for spatial awareness.

**Key technologies:** C# (.NET 8.0), tModLoader, Tolk library for universal screen reader support (JAWS, NVDA, Window-Eyes, SuperNova, System Access, ZoomText, SAPI fallback)

## Build Commands

```powershell
# Build and deploy to local tModLoader Mods folder
pwsh -NoProfile -ExecutionPolicy Bypass -File Tools/build.ps1

# Build only (no deployment)
pwsh -NoProfile -ExecutionPolicy Bypass -File Tools/build.ps1 -SkipDeploy

# Build with narration lint (checks client.log for NVDA failures)
pwsh -NoProfile -ExecutionPolicy Bypass -File Tools/build.ps1 -NarrationLint
```

The build script invokes tModLoader's build system (`dotnet tModLoader.dll -build`), not MSBuild directly. Output is `TerrariaAccess.tmod`.

When done with a task, build and deploy (without `-SkipDeploy`) so the mod is ready to test in tModLoader immediately.

## Architecture

### Core Systems (Mods/TerrariaAccess/Common/)

**Entry Point:** `TerrariaAccess.cs` - Initializes services and keybinds on mod load.

**Services Layer (`Services/`):**
- `ScreenReaderService` - Central speech API. Manages announcement categories (Default, Tile, Wall, Pickup, World) with per-category rate limiting. Routes to `SpeechController` -> `TolkSpeechProvider`.
- `SpatialAudioPanner` - Calculates stereo panning/pitch based on world position relative to player.
- `WorldAnnouncementService` - Handles world event announcements (blood moon, invasions, biome changes).
- `ChatHistoryService` - Chat message history tracking for review/narration.
- `ContextualInputRouter` - Routes keyboard input based on active UI context.
- `UiTickSoundPlayer` - Plays tick sounds on UI navigation changes.
- `SoundLoudnessUtility`, `SynthesizedSoundFactory` - Audio generation utilities.
- `UiAreaNarrationContext` - Tracks which UI area is active for narration context.
- `ScreenReaderDiagnostics` - Diagnostics/logging for Tolk communication.

**Systems Layer (`Systems/`):**
- `InGameNarrationSystem` - Partial class coordinating all in-game narrators via `NarrationScheduler`. Hooks into Terraria's ItemSlot, Main.NewText, PopupText, IngameOptions, etc.
- `MenuNarrationSystem` - Hooks into `Main.DrawMenu` to narrate main menu UI states.
- `GuidanceSystem` - Waypoint/target tracking with audio pings. Partial class split across:
  - `.cs` - Core logic, category cycling, waypoint management
  - `.Audio.cs` - Ping emission, tone generation
  - `.Scan.cs` - NPC/Player/Interactable/DroppedItem/Critter/Plantlife scanning
  - `.State.cs` - Selection mode state, waypoint storage
  - `.Networking.cs` - Multiplayer sync
  - Extracted helpers in `Guidance/` subdirectory: `GuidanceEntry`, `GuidanceScanner`, `GuidanceAudioPlayer`, `GuidanceStateManager`, `GuidanceNetworking`, `GuidanceKeybinds`

**Narrators (nested in InGameNarrationSystem):**
- `HotbarNarrator` - Hotbar slot navigation
- `InventoryNarrator` - Inventory navigation, split across partials:
  - `.Core.cs` - Main logic and slot focus tracking
  - `.Regions.cs` - Region detection and display names
  - `.Focus.cs`, `.Models.cs`, `.Tooltips.cs`, `.SpecialSelections.cs` - Supporting concerns
- `CraftingNarrator` - Recipe navigation, split across partials:
  - `.cs` - Core crafting UI handling
  - `.Guide.cs` - Guide menu and Goblin Tinkerer reforge
  - `.Recipe.cs` - Recipe types, resolution, and requirement building
- `CursorNarrator`, `SmartCursorNarrator` - Tile/cursor position narration
- `NpcDialogueNarrator` - NPC chat and shop interactions
- `ChatInputNarrator` - Chat text input narration
- `LockOnNarrator` - Lock-on target narration
- `WireColorMenuNarrator` - Wire color selection UI
- `IngameSettingsNarrator`, `ControlsMenuNarrator` - Settings UI
- `WorldInteractableTracker`, `TileToggleAnnouncer` - World interaction tracking
- Audio emitters: `FootstepAudioEmitter`, `ClimbAudioEmitter`, `BiomeAnnouncementEmitter`, `HostileStaticAudioEmitter`, `MultiplayerFootstepAudioEmitter`, `FootstepToneProvider`
- Descriptor services: `CursorDescriptorService`, `TileStateDescriptorService`, `WorldPositionalAudioService`

**Players (`Players/`):**
- `BuildModePlayer` - Keyboard-driven tile placement mode
- `DamageAnnouncementPlayer` - Combat damage narration
- `DeathNarrationPlayer` - Death event narration
- `NpcDialogueInputPlayer` - NPC dialogue input handling
- `GuidancePlayer` - Guidance system per-player state
- `StatusCheckPlayer` - Status check per-player state
- `WireColorMenuPlayer` - Wire color menu per-player state

**Build Mode (`Systems/BuildMode/`):**
- Provides keyboard-based cursor movement for placing/breaking tiles without mouse

**Gamepad Emulation (`Systems/GamepadEmulation/`):**
- Allows keyboard users to trigger gamepad-only UI navigation
- `DpadVirtualizationSystem` - Virtual D-pad from keyboard input
- `VirtualTriggerService`, `VirtualStickService` - Emulated gamepad controls

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChipsAhoiMcCoy/TerrariaAccess](https://github.com/ChipsAhoiMcCoy/TerrariaAccess) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
