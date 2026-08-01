---
trigger: always_on
description: You are an experienced Android app developer. Follow official Android architecture recommendations and [Nimble Compass](https://nimblehq.co/compass/) conventions.
---

# AGENTS.md

You are an experienced Android app developer. Follow official Android architecture recommendations and [Nimble Compass](https://nimblehq.co/compass/) conventions.

## Project Overview

Android app template implementing modern Android development best practices. This template serves as the foundation for Nimble's Android projects, demonstrating Clean Architecture with Jetpack Compose.

## Technology Stack

| Category | Technology |
|----------|------------|
| **Language** | Kotlin |
| **UI Framework** | Jetpack Compose with Material 3 |
| **Architecture** | MVVM / Clean Architecture |
| **State Management** | Unidirectional Data Flow (UDF) with StateFlow |
| **Dependency Injection** | Hilt |
| **Navigation** | Jetpack Navigation Compose |
| **Async** | Kotlin Coroutines + Flow |
| **Networking** | Retrofit + OkHttp + Moshi |
| **Local Storage** | DataStore, Room (when needed) |
| **Build System** | Gradle (Kotlin DSL) |
| **Min SDK** | 24 |
| **Target SDK** | 34 |
| **JVM** | Java 17 |

## Architecture

This project follows Google's official architecture guidance with a layered, modular structure:

- **UI Layer (`app/`):** Built entirely with Jetpack Compose. ViewModels act as state holders, exposing UI state as StateFlow streams.
- **Domain Layer (`domain/`):** Pure Kotlin module with business logic, models, and repository interfaces. No Android dependencies.
- **Data Layer (`data/`):** Repository implementations, API clients, and local storage. Depends on domain.

**Dependency Flow:** `app` → `domain` ← `data`

```
┌─────────────────────────────────────────────────────┐
│  app/                                               │
│  ├── di/          → Hilt modules                   │
│  ├── extensions/  → Kotlin extensions              │
│  ├── ui/                                            │
│  │   ├── base/    → BaseViewModel, BaseScreen      │
│  │   ├── screens/ → Feature screens & ViewModels   │
│  │   └── theme/   → AppColors, AppTypography, etc. │
│  └── MainApplication.kt                            │
├─────────────────────────────────────────────────────┤
│  domain/                                            │
│  ├── exceptions/  → Custom exceptions              │
│  ├── models/      → Domain entities                │
│  ├── repositories/→ Repository interfaces          │
│  └── usecases/    → Business logic                 │
├─────────────────────────────────────────────────────┤
│  data/                                              │
│  ├── local/       → DataStore, Room                │
│  ├── remote/      → API services, models, clients  │
│  └── repositories/→ Repository implementations     │
└─────────────────────────────────────────────────────┘
```

## Prerequisites

- Android Studio (Latest Stable or Preview for new features)
- JDK 17+
- API keys configured in `local.properties` (see README.md)

## Build Variants

The app has two product flavors and three build types:

| Flavor | Purpose |
|--------|---------|
| `staging` | Development/QA environment |
| `production` | Production environment |

| Build Type | Purpose |
|------------|---------|
| `debug` | Development with debug tools |
| `preRelease` | Release-like build with debug certificate |
| `release` | Optimized, ProGuard-enabled |

**Default development:** `assembleStagingDebug`

## Commands

### Build
```bash
./gradlew assembleStagingDebug
```

### Static Analysis
```bash
./gradlew detekt lint
```

### Tests
```bash
# All unit tests
./gradlew app:testStagingDebugUnitTest data:testDebugUnitTest domain:test

# Single test class
./gradlew app:testStagingDebugUnitTest --tests "co.nimblehq.template.compose.ui.screens.home.HomeViewModelTest"
```

### Coverage
```bash
./gradlew koverXmlReportCustom
```

## Testing

**Before commit:** `./gradlew detekt lint`

**Before PR:** `./gradlew detekt lint app:testStagingDebugUnitTest data:testDebugUnitTest domain:test koverXmlReportCustom`

### Test Frameworks

| Purpose | Library |
|---------|---------|
| **Mocking** | MockK |
| **Assertions** | Kotest matchers |
| **Flow testing** | Turbine |
| **Android testing** | Robolectric |
| **UI testing** | ComposeTestRule |

### Coverage Requirements
- **Project:** 80% minimum
- **Per-file:** 95% minimum

### Test Naming Convention
```kotlin
@Test
fun `When user taps login, it shows loading`() { }
```

## Key Files

| File | Purpose |
|------|---------|
| `app/.../MainActivity.kt` | Single activity entry point |
| `app/.../ui/AppNavGraph.kt` | Navigation graph |
| `app/.../ui/base/BaseViewModel.kt` | ViewModel base class |
| `data/.../remote/services/` | API service definitions |
| `detekt-config.yml` | Detekt rules |

## Configuration Files

| File | Purpose |
|------|---------|
| `detekt-config.yml` | Static analysis rules |
| `app/proguard-rules.pro` | ProGuard configuration |
| `local.properties` | API keys (gitignored) |

## Code Style

### Kotlin (Nimble Compass)
- **Indent:** 4 spaces (no tabs)
- **Line length:** 120 characters max
- **Class size:** 150 lines max
- **Method size:** 60 lines max
- **Parameters:** 5 max
- **Nested blocks:** 3 max
- **Boolean naming:** `is`/`has`/`can` prefix (e.g., `isValid`, `hasError`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nimblehq/android-templates](https://github.com/nimblehq/android-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
