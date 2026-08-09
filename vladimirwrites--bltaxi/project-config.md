---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BL Taxi is an Android application for quickly calling taxis in Banja Luka. It follows Clean Architecture principles with a multi-module setup, emphasizing testability, maintainability, and offline-first functionality.

## Build Commands

### Building the App
```bash
# Build debug variant
./gradlew assembleDebug

# Build release variant
./gradlew assembleRelease

# Install debug on connected device
./gradlew installDebug
```

### Running Tests
```bash
# Run all unit tests
./gradlew test

# Run tests for specific module
./gradlew :app:test
./gradlew :taxi:domain:test

# Run tests with coverage
./gradlew testDebugUnitTest

# Run single test class
./gradlew :app:testDebugUnitTest --tests "com.vlad1m1r.bltaxi.MainActivityShould"
```

### Code Quality
```bash
# Run detekt (static analysis)
./gradlew detekt

# Clean build
./gradlew clean
```

## Architecture Overview

### Multi-Module Structure

The project uses feature-based modularization following Clean Architecture:

```
app/                    # Main application module, navigation, DI setup
├── taxi/
│   ├── domain/        # Business logic, use cases, repository interfaces
│   ├── data/          # Repository implementations
│   └── ui/            # ViewModels, Fragments, adapters
├── about/
│   ├── domain/        # About feature business logic
│   ├── data/          # About data layer
│   └── ui/            # About UI components
├── settings/ui/       # Settings feature (UI only)
├── baseui/            # Shared UI components, binding adapters, navigation
├── basedata/          # Shared data utilities
├── local/             # Room database, SharedPreferences
├── remote/            # Retrofit API client
├── analytics/         # Firebase Analytics & Crashlytics
├── sync/              # WorkManager background sync
└── shortcuts/         # App shortcuts support
```

### Dependency Flow

Modules follow strict dependency rules:
- **UI** depends on → Domain + Data + BaseUI
- **Data** depends on → Domain + Remote/Local
- **Domain** depends on → Kotlin stdlib only (framework-agnostic)
- **Base modules** → Android framework only

### Key Architectural Patterns

1. **Clean Architecture**: Domain layer is framework-agnostic, Data layer acts as adapter
2. **MVVM**: ViewModels expose LiveData/ObservableBoolean, Fragments observe
3. **Repository Pattern**: Single source of truth, offline-first strategy (local cache → remote fallback)
4. **Use Cases**: Single-purpose classes with `invoke()` operator for business logic
5. **Dependency Injection**: Hilt with module-based organization
6. **Navigation Component**: Single-activity architecture with fragment navigation
7. **Sealed Classes**: Type-safe result handling (`TaxisResult.Success/Error`, `Action` types)

### Data Flow Example

Loading taxi list:
1. `TaxiFragment` observes `TaxiViewModel.taxis` LiveData
2. ViewModel calls `GetOrderedTaxiList` use case
3. Use case queries `TaxiRepository.getTaxis()`
4. Repository checks `TaxiProviderLocal` (Room) for cached data
5. If empty, fetches from `TaxiProviderRemote` (Retrofit) and caches locally
6. Returns sealed class result: `TaxisResult.Success(list)` or `TaxisResult.Error`
7. ViewModel maps domain models to presentation models on main dispatcher
8. Fragment updates RecyclerView

## Dependency Injection with Hilt

### Key Modules

- **AppModule** (`app/di/`): Navigator, CoroutineDispatcherProvider
- **LocalModule** (`local/di/`): Room database, SharedPreferences, OrderProvider
- **RemoteModule** (`remote/di/`): Retrofit client, API services
- **TaxiRepositoryModule** (`taxi/data/di/`): Repository implementations
- **AnalyticsModule** (`analytics/di/`): Firebase Analytics & Crashlytics
- **SyncModule** (`sync/di/`): WorkManager configuration
- **ShortcutModule** (`shortcuts/di/`): Shortcut handlers

### Scopes

- `@SingletonComponent`: App-wide singletons (repositories, database, API clients)
- `@ViewModelComponent`: ViewModel-scoped dependencies
- `@HiltViewModel`: Marks ViewModels for injection
- `@AndroidEntryPoint`: Activities/Fragments that receive injections

## Testing Strategy

### Test Types

The project uses Robolectric for Android-dependent unit tests, avoiding the need for instrumentation tests in most cases.

- **Unit Tests**: Domain logic, ViewModels, repositories (use Mockito, Truth assertions)
- **Fragment Tests**: Robolectric + FragmentScenario for UI testing
- **Test Utilities**: `HiltExt.kt` for Hilt test setup

### Running Specific Tests

```bash
# Test a specific ViewModel
./gradlew :taxi:ui:test --tests "*TaxiViewModelShould"

# Test with debug output
./gradlew test --info

# Test specific feature module
./gradlew :taxi:domain:test
```

## Version Catalog

Dependencies are managed in `gradle/libs.versions.toml`:
- All versions centralized in `[versions]` section
- Libraries defined in `[libraries]` section
- Plugins in `[plugins]` section
- Common dependency bundles in `[bundles]` section

When updating dependencies, modify `libs.versions.toml` only.

## Key Technical Details

### Language & Compilation
- Kotlin 2.0.20

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VladimirWrites/BLTaxi](https://github.com/VladimirWrites/BLTaxi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
