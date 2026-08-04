---
trigger: always_on
description: Generates `appdb.sqlite3` by:
---

# AGENTS.md

This file provides guidance to AI Agents (Codex, Copilot, Claude Code, etc.) when working with code in this repository.

## Project Overview

**Jyutping** is a Cantonese Input Method Editor (IME) for Android using the Hong Kong Linguistic Society's Jyutping romanization scheme. 

The project uses **Jetpack Compose** for UI and **Kotlin** for implementation. It includes a build-time data preparation module that generates the SQLite database used by the IME.

## Building and Development

### Build Environments and Tools
- Android Studio 2025.3.4+
- Android Gradle plugin 9.2.0+
- JDK 21
- Min SDK: 29 (Android 10)
- Target SDK: 37 (Android 17)
- Compile SDK: 37


### One-Time Setup: Prepare Databases

Before opening in Android Studio, prepare the input method databases:

```bash
cd ./preparing/
./gradlew run
```

This generates `appdb.sqlite3` which is copied to `app/src/main/assets/`. The database contains:
- 20+ tables with input lexicon, character data, and linguistic information
- Indexes for fast lookups during input
- Symbol, mark, emoji, and variant character tables

The preparing module is a standalone Gradle application that:
1. Loads .txt resource files (jyutping.txt, collocation.txt, etc.)
2. Creates normalized SQLite tables with proper indexes
3. Outputs the database to app assets

**Note**: Changes to resource files or database schema require re-running this step.

### Building the App

```bash
# Debug build
./gradlew :app:assembleDebug

# Release build
./gradlew :app:assembleRelease
```

### Running Tests

Tests are currently disabled (`tasks.withType<Test>().configureEach { enabled = false }`). To enable:
1. Remove the disable block in `app/build.gradle.kts`
2. Run: `./gradlew :app:testDebugUnitTest`
3. Run instrumented tests: `./gradlew :app:connectedAndroidTest`

## Architecture Overview

### Layered Design

```
Presentation (Compose UI)
  ↓ JyutpingInputMethodService (state holder + event handler)
  ↓ Business Logic (Segmenter, Researcher, Converter)
  ↓ Data Access (DatabaseHelper, InputMemoryHelper)
  ↓ SQLite Database
```

### Key Modules

#### **app/** - Main IME Application (20000+ LOC, 170+ Kotlin files)

**JyutpingInputMethodService.kt** - Core IME service
- Manages 60+ MutableStateFlow properties (input mode, candidates, buffer, settings)
- Handles soft keyboard input (VirtualInputKey enums) and physical keyboard events
- Processes input into candidates via Segmenter, Researcher, and Converter
- Records user memory in InputMemoryHelper for learning

**Package structure:**
- `keyboard/` - 30+ Compose components for different keyboard layouts and candidate display
- `models/` - Input/output data models, linguistic structures, and processing logic
- `utilities/` - Database access, shape mapping (stroke/cangjie), character conversion
- `editingpanel/` - Keyboard text editing buttons (copy/paste/cursor movement)
- `search/` - Part of the main app, linguistic lookup (definitions, collocations, homophones)
- `app/` - Main part of the main app, settings and educational screens (Cantonese, romanization guides)
- `ui/` - Compose theme and reusable components
- `ninekey/` - Nine-key (T9-style) keyboard layout
- `numeric/` - Numeric keyboard layout
- `stroke/` - Stroke input keyboard layout
- `shapes/` - Custom Compose bubble shapes for key rendering
- `emoji/` - Emoji keyboard
- `feedback/` - Audio and haptic feedback
- `extensions/` - Kotlin extension functions (Boolean, Char, String)
- `linguistics/` - IPA conversion and Old Cantonese data
- `presets/` - Preset constants, characters, colors, and strings
- `speech/` - Text-to-speech (TTS) support

#### **preparing/** - Build-Time Data Preparation (~1,600 LOC, 17 files)

Generates `appdb.sqlite3` by:
- **AppDataPreparer** - Creates linguistic reference tables (definitions, collocations, dictionaries)
- **KeyboardDataPreparer** - Creates input method tables (core lexicon, stroke/cangjie/pinyin data, variants)
- Database includes 25 tables and 56 indexes for performance

## Critical Data Flows

### Soft Keyboard Input Pipeline
```
User taps key → VirtualInputKey → bufferEvents (observable)
  → Determine input mode (Cantonese/Pinyin/Stroke/etc.)
  → Segmenter.segment() → Researcher.suggest()
  → Converter.dispatch() (merging + sorting)
  → candidates StateFlow → CandidateBoard renders
```

### Physical Keyboard Input
- Hardware key events → `onKeyDown()/onKeyUp()` → `handlePhysicalKeyEvent()`
- Key codes are mapped to VirtualInputKeys within the service
- Can trigger full CandidateBoard or inline PhysicalKeyboardCandidateBar
- Numbers 1-9 and 0 select candidates 1-10 respectively; Tab cycles through groups of 10

### Candidate Selection & User Memory
- User selects candidate → `selectCandidate()`
- Commits text via InputConnection
- Records in InputMemoryHelper (separate database) for future ranking
- Clears buffer if input is complete

## Database Schema

**Main Database: appdb.sqlite3**

Core input tables:
- `core_lexicon` - Main word database
- `structure_table` - Character composition (stroke/shape codes)
- `syllable_table`, `pinyin_syllable_table` - Syllable mappings
- `pinyin_lexicon` - Pinyin input lexicon
- `stroke_table`, `cangjie_table`, `quick_table` - Shape input methods

Linguistic reference tables:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuetyam/jyutping-android](https://github.com/yuetyam/jyutping-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
