---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

RimWorld Access is a C# mod for RimWorld that provides screen reader accessibility. It uses Harmony patches to inject keyboard navigation into RimWorld's UI and the Prism library for cross-platform screen reader integration (NVDA, JAWS, VoiceOver, Orca, SAPI, and more).

**Technology Stack:**
- .NET Framework 4.7.2
- HarmonyLib 2.3.3 (runtime patching)
- Prism (screen reader integration via P/Invoke — supports Windows, macOS, Linux)
- RimWorld 1.6 assemblies

## Building and Testing
### Build Commands

```bash
# Build and auto-deploy to RimWorld/Mods/RimWorldAccess/
dotnet build

# Build release package to release/RimWorldAccess/
dotnet build -c Release

# Update Prism native libraries to latest release
dotnet msbuild -t:UpdatePrism
```

**Build Output:**
- DLL: `bin/Debug/net472/rimworld_access.dll`
- Auto-deploys to: `$(RimWorldDir)\Mods\RimWorldAccess\Assemblies\`
- Native Prism libraries (prism.dll / libprism.dylib / libprism.so) copied to mod root

## Code Architecture

### Core Pattern: State + Patch

Every feature follows this architecture:

1. **State class** (`*State.cs`)
   - Maintains navigation state (selected index, list of items, etc.)
   - `IsActive` flag checked by UnifiedKeyboardPatch
   - Methods: `Open()`, `Close()`, `SelectNext()`, `SelectPrevious()`
   - Calls `TolkHelper.Speak()` to announce selections

2. **Patch class** (`*Patch.cs`)
   - Harmony patches that intercept RimWorld methods
   - Initializes State when UI opens (PostOpen/Postfix)
   - Resets State when UI closes
   - May inject accessibility into rendering code

3. **Helper class** (`*Helper.cs`)
   - Data extraction utilities
   - Reusable functions for formatting announcements
   - No state management

### Module Organization

The codebase is organized into 18 modules by game feature:

| Module | Files | Purpose |
|--------|-------|---------|
| **Core/** | 2 | Mod entry point, Harmony initialization |
| **ScreenReader/** | 5 | Prism screen reader integration and audio |
| **Input/** | 1 | UnifiedKeyboardPatch - central input router |
| **MainMenu/** | 19 | Main menu and game setup flow |
| **Map/** | 9 | Map navigation, cursor, scanner |
| **World/** | 8 | World map, settlements, caravans |
| **Building/** | 23 | Construction, zones, areas |
| **Inspection/** | 24 | Building/object inspection UI, Info Card |
| **Pawns/** | 25 | Pawn info and character tabs |
| **Work/** | 2 | Work priorities and schedules |
| **Animals/** | 6 | Animal and wildlife management |
| **Prisoner/** | 3 | Prisoner management |
| **Quests/** | 3 | Quests and notifications |
| **Combat/** | 2 | Combat and targeting |
| **Trade/** | 3 | Trading system |
| **Research/** | 2 | Research system |
| **UI/** | 13 | Generic dialogs and windowless menus |

Each module has its own `CLAUDE.md` with detailed documentation.

### Central Systems

**UnifiedKeyboardPatch** (`Input/UnifiedKeyboardPatch.cs`)
- Central keyboard input router for ALL accessibility features
- Patches `UIRoot.UIRootOnGUI` at Prefix level
- Priority system (lower number = higher priority, range -1 to 10)
- Checks `IsActive` flags before routing to State classes
- Calls `Event.current.Use()` to consume events and prevent default game behavior

**TolkHelper** (`ScreenReader/TolkHelper.cs`)
- Cross-platform screen reader integration via Prism library
- `TolkHelper.Speak(text, priority)` used by all modules
- Three priorities: Low (don't interrupt), Normal, High (interrupt)
- Backed by: `NativeLibraryLoader.cs` (cross-platform DLL loading) and `PrismNative.cs` (Prism C API bindings)
- Initialized in `Core/rimworld_access.cs`

**MapNavigationState** (`Map/MapNavigationState.cs`)
- Provides `CurrentCursorPosition` (IntVec3) used by 10+ modules
- Arrow key navigation with camera follow
- Jump modes for terrain features

### Dependency Graph

```
Core/rimworld_access.cs (entry point)
  └── ScreenReader/TolkHelper (initialize)
        └── Input/UnifiedKeyboardPatch (routes to all modules)
              ├── MainMenu/
              ├── Map/ → [Building, Inspection, Quests, Combat]
              ├── Pawns/ → [Work, Prisoner]
              ├── World/ → [Quests]
              ├── Animals/
              ├── Trade/
              ├── Research/
              └── UI/ → [All modules]
```

## Common Development Tasks

### Adding a New Feature

1. **Choose module directory** (or create new one under `src/`)
2. **Create State class:**
   ```csharp
   public static class MyFeatureState
   {
       public static bool IsActive { get; set; }
       private static int selectedIndex = 0;

       public static void Open()
       {
           IsActive = true;
           TolkHelper.Speak("Feature opened", SpeechPriority.Normal);
       }

       public static void Close()
       {
           IsActive = false;
       }
   }
   ```

3. **Create Patch class:**
   ```csharp
   [HarmonyPatch(typeof(RimWorldClass))]
   [HarmonyPatch("MethodName")]
   public static class MyFeaturePatch
   {
       [HarmonyPostfix]
       public static void Postfix()
       {
           MyFeatureState.Open();
       }
   }
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaronr7734/rimworld_access](https://github.com/aaronr7734/rimworld_access) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
