---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ComposeVisibilitySample is a Kotlin Multiplatform (Android + iOS) demo app showcasing Jetpack Compose's `onVisibilityChanged` API. The app displays images from Lorem Picsum API in a scrollable list and logs visibility events when an image is 50% visible for 1+ second.

**Key Documentation:**
- `docs/requirements.md` - Functional requirements, API specs, log format
- `docs/architecture.md` - Detailed architecture, testing strategy, implementation status
- `README.md` - Quick start guide

## Commands

### Build & Run
```bash
# Android
./gradlew :composeApp:assembleDebug
./gradlew :composeApp:installDebug

# iOS framework
./gradlew :composeApp:linkDebugFrameworkIosSimulatorArm64
```

### Testing
```bash
# Run all tests (39 tests total)
./gradlew :composeApp:testDebugUnitTest

# Run tests with output
./gradlew :composeApp:testDebugUnitTest --rerun-tasks

# Run specific test class
./gradlew :composeApp:testDebugUnitTest --tests "dev.dai.compose.visibility.sample.ui.viewmodel.ImageListViewModelTest"

# Run single test
./gradlew :composeApp:testDebugUnitTest --tests "dev.dai.compose.visibility.sample.ui.viewmodel.ImageListViewModelTest.uiState is Success when images are loaded successfully"
```

### Other Commands
```bash
# View dependencies
./gradlew :composeApp:dependencies

# Compile common Kotlin metadata
./gradlew :composeApp:compileCommonMainKotlinMetadata

# Clean build
./gradlew clean
```

## Architecture

This project follows Android Architecture Guidelines with strict layer separation:

### Three-Layer Architecture

```
UI Layer (commonMain/ui)
    ↓ StateFlow
Domain Layer (commonMain/domain)
    ↓ Result<T>
Data Layer (commonMain/data)
```

**Critical Design Patterns:**

1. **Unidirectional Data Flow (UDF)**: ViewModel exposes `StateFlow<ImageListUiState>`, UI observes and reacts
2. **Result Type for Error Handling**: All repository/use case methods return `Result<T>`, never throw exceptions upward
3. **Dependency Injection with Koin**: All dependencies managed through `di/AppModule.kt`
4. **expect/actual for Platform-Specific Code**: `HttpClientFactory` uses expect/actual pattern
   - `commonMain/di/HttpClientFactory.kt` (expect declaration)
   - `androidMain/di/HttpClientFactory.android.kt` (OkHttp engine)
   - `iosMain/di/HttpClientFactory.ios.kt` (Darwin engine)

### Key Architectural Rules

- **Domain Models are Platform-Independent**: `ImageItem`, `VisibilityLog`, `MicrosecondTimestamp` in `domain/model`
- **DTOs Only in Data Layer**: `ImageResponse` in `data/model`, converted via `toDomainModel()` extension
- **Use Cases are Mandatory**: All business logic must go through use cases, never call repositories directly from ViewModel
- **ViewModel is Shared**: `ImageListViewModel` in `commonMain` shared between Android and iOS

### Package Structure

```
composeApp/src/
├── commonMain/kotlin/dev/dai/compose/visibility/sample/
│   ├── ui/          # Composables, ViewModels
│   │   ├── screen/      # ImageListScreen
│   │   ├── component/   # ImageCard, ErrorView, LoadingView
│   │   └── viewmodel/   # ImageListViewModel
│   ├── domain/      # Use Cases, Models, Repository Interfaces
│   │   ├── model/       # ImageItem, VisibilityLog, MicrosecondTimestamp
│   │   ├── repository/  # ImageRepository (interface)
│   │   └── usecase/     # GetImagesUseCase, LogImageVisibilityUseCase
│   ├── data/        # Repository Implementations, Data Sources, DTOs
│   │   ├── repository/  # ImageRepositoryImpl
│   │   ├── remote/      # RemoteDataSource, PicsumRemoteDataSource
│   │   └── model/       # ImageResponse (DTO)
│   └── di/          # AppModule (Koin), HttpClientFactory (expect/actual)
│
├── commonTest/      # Domain/Data layer tests (Kotlin Test)
│   ├── domain/
│   ├── data/
│   └── fake/        # FakeImageRepository, FakeRemoteDataSource
│
└── androidUnitTest/ # ViewModel tests (JUnit + MainDispatcherRule)
    ├── ui/viewmodel/
    └── util/        # MainDispatcherRule
```

## Testing Strategy

### Test Distribution
- **commonTest** (30 tests): Domain models, use cases, repositories (uses Kotlin Test)
- **androidUnitTest** (9 tests): ViewModel (uses JUnit + MainDispatcherRule)
- **Total**: 39 tests, 80%+ coverage

### Testing Patterns
1. **Fake Pattern Preferred**: Use `FakeImageRepository`, `FakeRemoteDataSource` from `commonTest/fake`
2. **MainDispatcherRule for ViewModel Tests**: Always use `@get:Rule val mainDispatcherRule = MainDispatcherRule()` in ViewModel tests
3. **No MockK in Common Code**: MockK only for platform-specific classes in `androidUnitTest`

### Test File Locations
- Domain/Data tests: `commonTest/` (can run on all platforms)
- ViewModel tests: `androidUnitTest/` (requires JUnit, MainDispatcherRule)

## onVisibilityChanged Implementation

The core feature uses `Modifier.onVisibilityChanged` in `ImageCard.kt`:

```kotlin
Modifier.onVisibilityChanged(
    minFractionVisible = 0.5f,  // 50% threshold
    minDurationMs = 1000L,       // 1 second duration
    onVisibilityChanged = { imageId, position ->
        viewModel.onImageVisible(imageId, position)
    }
)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dai1678) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
