---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

This is the **Blitzortung Lightning Monitor** Android application - a real-time lightning visualization app built in Kotlin that displays lightning strike data from the blitzortung.org network. The app features map-based strike visualization, proximity alerts, background monitoring, and localization support for 10+ languages.

Project documentation: https://blitzortung.tryb.de

## Build Commands

### Building the app
```bash
./gradlew build
```

### Running tests
```bash
# Run all unit tests
./gradlew testDebugUnitTest

# Run specific test class
./gradlew testDebugUnitTest --tests "org.blitzortung.android.alert.AlertResultTest"

# Run tests with coverage report
./gradlew testDebugUnitTest jacocoTestReport
# Coverage report: app/build/reports/jacoco/jacocoTestReport/html/index.html
```

### Linting and code quality
```bash
# Run lint
./gradlew lint

# Run SonarQube analysis (requires SONAR_TOKEN)
./gradlew sonar

# Update the verification metadata
./gradlew --write-verification-metadata sha256 help
```

### Building release artifacts
```bash
# Build APK
./gradlew assembleRelease

# Build Android App Bundle (AAB)
./gradlew bundleRelease
```

### Installing on device
```bash
# Install debug build
./gradlew installDebug

# Install and launch
adb install -r app/build/outputs/apk/debug/app-debug.apk
adb shell am start -n org.blitzortung.android.app/.app.Main
```

## Technology Stack

- **Language**: Kotlin 2.2.10, JVM 17
- **Min SDK**: 21, Target SDK: 35
- **Build System**: Gradle 8.13
- **DI Framework**: Dagger 2.57.1
- **Testing**: JUnit 4, AssertJ, MockK, Robolectric
- **Map Library**: OSMDroid 6.1.20
- **UI**: ViewBinding, Material Components, AndroidX libraries

## Code Architecture

### High-Level Structure

The app follows a **component-based architecture** with Dagger 2 dependency injection coordinating the major subsystems:

```
BOApplication (Dagger root)
    ├── Main Activity (foreground UI)
    │   ├── MapFragment with overlays
    │   ├── UI components (status, alerts, histogram, etc.)
    │   └── Controllers (history, notifications, buttons)
    └── AppService (background monitoring)
        └── Minimal UI (notifications only)

Shared Singletons:
    ├── MainDataHandler / ServiceDataHandler (data flow)
    ├── LocationHandler (GPS tracking)
    ├── AlertHandler (proximity alerts)
    └── DataProviderFactory (API clients)
```

### Dagger 2 Dependency Injection

**Component**: `AppComponent` (singleton scope)

**Modules**:
- `AppModule`: Application context, SharedPreferences, PackageInfo, NotificationManager, Vibrator, WakeLock
- `ServiceModule`: Handler (main thread), AlarmManager, Period (timing utilities)
- `ActivityBindingModule`: Auto-injects Main activity and AppService
- `SettingsModule`: Auto-injects SettingsFragment

All major components (`@Inject` annotated) are wired through Dagger. The `BOApplication` class initializes the component graph on startup.

### Event-Driven Communication (ConsumerContainer Pattern)

The app uses a **publish-subscribe pattern** via `ConsumerContainer<T>` for component communication:

**Event Types**:
- `DataEvent`: Strike data updates (ResultEvent, RequestStartedEvent, StatusEvent)
- `AlertEvent`: Alert state changes (AlertResultEvent, AlertCancelEvent)
- `LocationEvent`: GPS location updates

**Key Producers**:
- `MainDataHandler` / `ServiceDataHandler`: Broadcasts `DataEvent` when new strike data arrives
- `AlertHandler`: Broadcasts `AlertEvent` when proximity alerts trigger/cancel
- `LocationHandler`: Broadcasts `LocationEvent` when device location changes

**Consumers** register via `requestUpdates(consumer: (Event) -> Unit)` and receive callbacks when events are broadcast. The last event is cached and immediately sent to new consumers.

### Data Flow Architecture

#### Data Providers (Strategy Pattern)

Two interchangeable data provider implementations:

1. **JsonRpcDataProvider** (default): JSON-RPC over HTTP with incremental updates
2. **BlitzortungHttpDataProvider**: HTTP GET with GZIP compression, requires auth

Factory: `DataProviderFactory` selects provider based on user preference

#### MainDataHandler (Data Orchestrator)

- Manages data fetch lifecycle and coordinates updates
- **Caching**: `DataCache` with 5-minute TTL to avoid redundant API calls
- **Sequencing**: `SequenceValidator` prevents stale data from overwriting fresh data
- **Animation mode**: Special handling for historical playback
- Responds to location changes by updating grid parameters
- Publishes `DataEvent` to all registered consumers

**Typical flow**: Timer triggers → updateData() → check cache → FetchDataTask (background) → API call → cache result → broadcast ResultEvent → consumers update UI/alerts

### Alert System Architecture

**AlertHandler**: Coordinates alert monitoring

- Subscribes to `DataEvent` (strike updates) and `LocationEvent` (position changes)
- Delegates computation to `AlertDataHandler`
- Broadcasts `AlertEvent` to UI components and `AlertSignal` (vibration/sound)

**AlertDataHandler**: Core alert logic

- Divides 360° into sectors (default 8: N, NE, E, SE, S, SW, W, NW)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wuan/bo-android](https://github.com/wuan/bo-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
