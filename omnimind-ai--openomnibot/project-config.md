---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

OmnibotApp is an AI-powered intelligent robot assistant application for Android. It's a hybrid app combining native Android Kotlin code with Flutter UI, implementing a modular architecture with clear separation of concerns.

**Key characteristics:**
- Android app with embedded Flutter UI module
- Modular monorepo architecture with feature-specific modules
- State machine-based task management system
- Accessibility services and overlay functionality
- AI/ML intelligence integration (on-device models)

## Build and Development Commands

### Android/Gradle Commands
```bash
# Full project build
./gradlew build

# Build debug APK (develop flavor)
./gradlew assembleDevelopDebug

# Build release APK (production flavor)
./gradlew assembleProductionRelease

# Run tests
./gradlew test

# Run instrumented tests
./gradlew connectedAndroidTest

# Lint checking
./gradlew lint

# Install debug APK to connected device
./gradlew installDevelopDebug
```

### Flutter Commands (for ui/ module)
```bash
cd ui

# Install dependencies
flutter pub get

# If you encounter "Could not read script '.../ui/.android/include_flutter.groovy'" error:
flutter clean
flutter pub get

# Build Flutter module as AAR
flutter build aar

# Run Flutter tests
flutter test

# Analyze Flutter code
flutter analyze
```

### Project Setup
The project requires importing the OmniIntelligence module as an external module:
```bash
# Clone the companion repository
git clone https://github.com/omnimind-ai/OmniIntelligence

# Import OmniIntelligence/OmniIntelligence as a module in Android Studio
```

## Architecture Overview

### Module Structure
```
OmnibotApp/
├── app/                 # Main application module (entry point, activities)
├── ui/                  # Flutter UI module (cross-platform UI with Riverpod)
├── baselib/             # Core libraries (database, networking, auth, storage)
├── assists/             # Task management and state machine
├── omniintelligence/    # AI/ML intelligence modules (external import)
├── overlay/             # Floating overlay functionality
├── accessibility/       # Accessibility services for UI automation
└── testbot/             # Testing utilities (develop flavor only)
```

### Core Architectural Patterns

**1. State Machine Pattern** (`assists/StateMachine.kt`)
- Central task lifecycle management (Companion, Learning, Scheduled tasks)
- Coordinates state transitions between different task types
- Manages communication between UI, services, and background tasks

**2. Flutter-Native Embedding**
- Flutter module embedded in native Android app via `FlutterEngineGroup`
- Communication channels between Kotlin and Flutter
- Shared resource management across Flutter engine instances

**3. Task-Based System**
- Three task types: Companion, Learning, Scheduled
- Task parameters and result callbacks
- Background execution with Kotlin coroutines

**4. Service-Oriented Architecture**
- Accessibility services for interaction monitoring
- Overlay services for floating UI elements
- Background services for long-running tasks

### Key Integration Points

**Assists Module** (`assists/`)
- `StateMachine.kt`: Core state machine managing task lifecycles
- `AssistsCore.kt`: SDK interface for task creation, state changes, and results
- `CompanionController.kt`: Interface for companion mode tasks (engineering team)
- `TaskFilterServer.kt`: XML-based scene filtering and matching (research team)

Directory structure:
- `api/`: Models, enums, listeners
- `controller/`: Controllers providing functionality for tasks
- `server/`: Core services for XML acquisition and scene filtering
- `task/`: Core task modules (Companion, Scheduled, Learning tasks)
- `util/`: Utility classes

**Database Layer** (`baselib/`)
- Room database with DAOs for conversations and messages
- MMKV for lightweight key-value storage
- Located in `baselib/src/main/java/cn/com/omnimind/baselib/database/`

**Flutter UI** (`ui/`)
- Riverpod for state management
- Go Router for navigation
- Material Design 3 components
- Embedded as AAR module in native app

## Build Flavors

The project uses product flavors for different environments:

**develop**: Development environment
- Optional backend via `OMNIBOT_BASE_URL` (empty by default in open-source mode)
- Includes testbot module
- Debug signing config (Android default debug keystore)

**production**: Production environment
- Optional backend via `OMNIBOT_BASE_URL` (empty by default in open-source mode)
- Excludes testbot module
- Release signing config with V2/V3 signatures

## Configuration

Optional/required properties in `gradle.properties` or `~/.gradle/gradle.properties`:

```properties
# Optional backend endpoint for self-hosted deployments
OMNIBOT_BASE_URL=

# Required only for release signing
OMNI_RELEASE_STORE_FILE=/abs/path/release.jks
OMNI_RELEASE_STORE_PWD=***
OMNI_RELEASE_KEY_ALIAS=***
OMNI_RELEASE_KEY_PWD=***
```

## Development Notes

### Platform Requirements
- **Min SDK**: 30 (Android 11)
- **Target SDK**: 34 (Android 14)
- **Compile SDK**: 36
- **NDK**: ARMv7 and ARM64 architectures
- **JDK**: 11+
- **Flutter**: 3.9.2+

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omnimind-ai/OpenOmniBot](https://github.com/omnimind-ai/OpenOmniBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
