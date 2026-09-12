---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**AutoAction** is an Android automation app that uses AccessibilityService to enable script-based screen interactions (clicks, swipes, multi-touch) without requiring root access. The app targets gaming scenarios (particularly MOBA games) where users need rapid, repeatable macro operations.

### Core Capabilities
- Visual script recording via transparent overlay capture
- Script library management with independent floating shortcuts
- Atomic action model (CLICK, SWIPE, DELAY) with randomization support
- Multi-instance floating windows for quick macro execution

## Build Commands

Navigate to the `app/` directory for all Gradle commands:

```bash
cd app/

# Build the app
./gradlew assembleDebug

# Install debug build to connected device
./gradlew installDebug

# Run tests
./gradlew test                    # Unit tests
./gradlew connectedAndroidTest    # Instrumented tests

# Clean build
./gradlew clean

# Lint check
./gradlew lint

# Build release APK
./gradlew assembleRelease
```

## Architecture

### Technology Stack
- **Language**: Kotlin
- **UI Framework**: Jetpack Compose (modern screens) + XML ViewBinding (legacy fragments)
- **Architecture**: MVVM pattern
- **Database**: Room (local SQLite)
- **DI**: Manual dependency injection via service singletons
- **Async**: Coroutines with Flow for reactive data streams

### Key Components

#### 1. AccessibilityService Layer
**`AutoActionService.kt`** - Core accessibility service
- Implements `AccessibilityService` to dispatch gestures
- Maintains singleton instance accessible via `getInstance()`
- Hosts `ScriptExecutor` for running automation scripts
- Communicates service status via `StateFlow`

**`ScriptExecutor.kt`** - Script orchestration
- Manages script lifecycle (start, loop, stop)
- Delegates gesture execution to `GestureExecutor`
- Applies global randomization settings per-script
- Handles loop modes: infinite (count=0) or finite (count=N)

**`GestureExecutor.kt`** - Low-level gesture dispatch
- Translates `Action` data classes into Android `AccessibilityNodeInfo.GestureDescription`
- Applies coordinate offset and duration variance for anti-detection

#### 2. Floating Window System
**`FloatingWindowService.kt`** - Overlay manager
- Creates two types of floating UI:
  - **Control Bar**: Global controls (recording, settings, shortcut toggle)
  - **Script Shortcuts**: Individual floating buttons per enabled script
- Uses `WindowManager` with `TYPE_APPLICATION_OVERLAY`
- Implements touch disambiguation: <40px movement = click, ≥40px = drag
- Persists shortcut positions to database on drag completion

**`RecordingService.kt`** - Gesture capture
- Overlays full-screen transparent layer to intercept `MotionEvent`
- Converts touch sequences into `Action` objects with calculated delays
- Blocks underlying app interaction during recording (standard Android limitation)

#### 3. Data Layer
**Room Database** (`AppDatabase.kt`)
- Single entity: `ScriptEntity` (JSON-serialized fields for `actions`, `shortcutConfig`)
- DAO operations exposed as `Flow` for reactive UI updates

**Data Models** (in `data/model/`)
- `Script`: Domain model containing action list, loop config, randomization params
- `Action`: Atomic instruction with type (CLICK/SWIPE/DELAY), coordinates, duration
- `ShortcutConfig`: Floating button appearance (icon, position, alpha, scale)
- `ActionType`: Enum defining supported gesture primitives

**Settings Layer** (`data/settings/`)
- `GlobalSettings`: Data class for app-wide randomization parameters
- `SettingsRepository`: DataStore-backed persistence for global settings

**Repository** (`ScriptRepository.kt`)
- Abstracts database access
- Provides `enabledScripts` Flow for observing active shortcuts
- Handles entity ↔ domain model conversion

#### 4. UI Layer
**Compose Screens** (`ui/screen/`)
- `ScriptListScreen`: Main list with accessibility service status indicator
- `ScriptEditorScreen`: Visual action sequence editor with drag-to-reorder
- `SettingsScreen`: Global randomization and anti-detection settings

**Navigation** (`ui/navigation/NavGraph.kt`)
- Simple Compose Navigation with three routes:
  - `script_list` (home)
  - `script_editor/{scriptId}` (create/edit)
  - `settings` (global settings)

**ViewBinding Fragments** (legacy, in `ui/home/`, `ui/dashboard/`, etc.)
- Older XML-based fragments for settings/notifications
- Coexist with Compose screens during migration

### Critical Implementation Details

#### Action Model v2.0 (Atomic Design)
The original design coupled clicks with delays (`baseDelay` field). **v2.0 refactored** to atomic actions:
- Old: `CLICK(x=100, y=200, baseDelay=1000)`
- New: `CLICK(x=100, y=200, duration=50)` + `DELAY(duration=1000)`

This enables advanced patterns like "continuous taps without pause" or "hold button indefinitely."

#### Randomization System (Anti-Detection) - v2.0 Implementation
**Global settings** (stored in DataStore via `SettingsRepository`):
- `randomization_enabled`: Master toggle (default: false)
- `click_offset_radius`: ±N pixels from target coordinates (default: 10px, range: 0-50)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ld1856/AutoAction](https://github.com/ld1856/AutoAction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
