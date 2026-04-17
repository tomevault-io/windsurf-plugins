---
trigger: always_on
description: HandleSignal<MySignal> { signal ->
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Anchor is a **state management architecture** for Kotlin Multiplatform applications built on Unidirectional Data Flow (UDF) design with Jetpack Compose integration. It targets Android, iOS (via iosX64, iosArm64, iosSimulatorArm64), and Desktop (JVM).

**Core philosophy**: Minimal, type-safe state management using receiver-based DSLs, immutable state updates, and Flow-based reactivity.

## Build Commands

### Testing

```bash
# Run all tests across all platforms and generate aggregated report
./gradlew allTests

# Run tests for specific platforms
./gradlew desktopTest              # Desktop/JVM tests
./gradlew testAndroid               # Android unit tests (all variants)
./gradlew testDebugUnitTest         # Android debug unit tests only
./gradlew iosSimulatorArm64Test     # iOS simulator tests
./gradlew iosX64Test                # iOS x64 tests

# Run connected device tests (requires connected Android device/emulator)
./gradlew connectedAndroidTest      # All flavors
./gradlew connectedDebugAndroidTest # Debug build only
./gradlew connectedAndroidDeviceTest # Tests for androidMain on connected devices

# Run single test class (example for JVM)
./gradlew :anchor-test:desktopTest --tests "dev.kioba.anchor.test.CounterTest"
```

### Building

```bash
# Build all modules
./gradlew build

# Build specific modules
./gradlew :anchor:build           # Core library
./gradlew :anchor-test:build      # Test utilities
./gradlew :androidApp:build       # Sample Android app

# Assemble Android app
./gradlew :androidApp:assembleDebug
./gradlew :androidApp:assembleRelease
```

### Code Quality

```bash
# Run lint on default variant
./gradlew lint

# Run lint with auto-fix for safe suggestions
./gradlew lintFix

# Update lint baseline
./gradlew updateLintBaseline

# Run all checks (tests + lint)
./gradlew check
```

### Publishing

```bash
# Publish to Maven Central (requires credentials)
./gradlew publishToMavenCentral

# Publish to GitHub Packages (requires GPR credentials)
./gradlew publish
```

**Note**: Publishing requires credentials set via:
- `gpr.user` and `gpr.key` properties (GitHub Packages)
- Maven Central credentials via vaniktechMavenPublish plugin

## Architecture

### Core Abstractions

The architecture is built on a hierarchy of capabilities provided through interfaces:

```
Anchor<E, S>  (abstract base)
├── StateAnchor<S>          - Read-only state access via `state: StateFlow<S>`
├── MutableStateAnchor<S>   - State mutations via `reduce { copy(...) }`
├── EffectAnchor<E>         - Side effect execution with effect scope
├── CancellableAnchor<E,S>  - Cancellable task management (keyed jobs)
├── SubscriptionAnchor      - Event emission via `emit { ... }`
└── SignalAnchor            - One-time signals via `post { ... }`
    └── AnchorSink<E, S>    - Combines all above capabilities
        └── AnchorRuntime<E, S>  - Concrete implementation
```

**Implementation**: `AnchorRuntime` (`anchor/src/commonMain/kotlin/dev/kioba/anchor/AnchorRuntime.kt`) manages:
- State via `MutableStateFlow<S>`
- Signals via `MutableSharedFlow<SignalProvider>` (one-time UI events)
- Events via `MutableSharedFlow<Event>` (internal reactive stream)
- Coroutine jobs via `MutableMap<Any, Job>` (cancellable operations)

### Marker Interfaces

Four empty marker interfaces provide type safety (`anchor/src/commonMain/kotlin/dev/kioba/anchor/AnchorMarkers.kt`):

- `ViewState` - UI state data classes
- `Effect` - External dependencies/side effect scope
- `Signal` - One-time notifications (toasts, navigation, etc.)
- `Event` - Internal event stream for subscriptions

### Data Flow Pattern

```
User Action (Button Click)
    ↓
anchor(CounterAnchor::increment)  // Type-safe callback via CompositionLocal
    ↓
AnchorScope.execute { anchor.increment() }  // Coroutine launch (Dispatchers.Default)
    ↓
reduce { copy(count = count.inc()) }  // Immutable state update
    ↓
MutableStateFlow.update()  // Flow emission
    ↓
collectAsState() in RememberAnchor  // Recomposition trigger
    ↓
UI Recomposition
```

**Parallel flows**:
- `post { Signal }` → `MutableSharedFlow<SignalProvider>` → `HandleSignal` → LaunchedEffect
- `emit { Event }` → `MutableSharedFlow<Event>` → Subscription chains
- `effect { }` → Coroutine execution with E (Effect) receiver

### Compose Integration

**Primary composable**: `RememberAnchor` (`anchor/src/androidMain/kotlin/dev/kioba/anchor/compose/RememberAnchor.kt`)

Responsibilities:
1. Creates/retrieves ViewModel-scoped `AnchorRuntime` via `ContainerViewModel`
2. Collects `state: StateFlow<S>` as `State<S>` (Main.immediate dispatcher)
3. Collects `signals: StateFlow<SignalProvider>` as `State<SignalProvider>`
4. Provides `AnchorScope` via `LocalAnchor` CompositionLocal
5. Provides `SignalProvider` via `LocalSignals` CompositionLocal

**Action creation**: Four overloaded `anchor()` composables (`AnchorAction.kt`) create type-safe callbacks:
```kotlin
anchor(CounterAnchor::increment)              // () -> Unit
anchor(CounterAnchor::updateById)             // (Int) -> Unit
anchor(CounterAnchor::updateByIdAndName)      // (Int, String) -> Unit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kioba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
