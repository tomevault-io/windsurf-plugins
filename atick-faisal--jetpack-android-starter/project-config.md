---
trigger: always_on
description: This file provides AI coding agents with project-specific instructions, conventions, and boundaries for working with this Android codebase.
---

# Jetpack Android Starter - AI Agent Instructions

This file provides AI coding agents with project-specific instructions, conventions, and boundaries for working with this Android codebase.

## Essential Commands

### Build & Run
```bash
# Build debug variant
./gradlew assembleDebug

# Build and install on connected device
./gradlew installDebug

# Clean build artifacts
./gradlew clean

# Build release (requires keystore.properties)
./gradlew assembleRelease
```

### Code Quality (ALWAYS RUN BEFORE COMMITTING)
```bash
# Auto-format all code with ktlint
./gradlew spotlessApply

# Check formatting
./gradlew spotlessCheck

# Run all checks
./gradlew check
```

### Testing
```bash
# Run unit tests
./gradlew test

# Run tests for specific module
./gradlew :feature:home:test

# Run instrumentation tests
./gradlew connectedAndroidTest
```

### Documentation
```bash
# Generate API documentation with Dokka
./gradlew dokkaHtmlMultiModule
# Output: build/dokka/htmlMultiModule/
```

### Firebase Setup
```bash
# Get SHA-1 fingerprint for Firebase console
./gradlew signingReport
```

## Project Context

### Tech Stack Overview
- **Language**: Kotlin 2.3.20 with coroutines & Flow
- **UI**: Jetpack Compose with Material3 (declarative UI)
- **Architecture**: Two-layer MVVM (UI + Data, intentionally no Domain layer)
- **DI**: Dagger Hilt (compile-time injection)
- **Local Storage**: Room (SQL) + DataStore (key-value)
- **Networking**: Retrofit + OkHttp + Kotlinx Serialization
- **Backend**: Firebase (Auth, Firestore, Crashlytics, Performance)
- **Background Work**: WorkManager with sync constraints
- **Build**: Gradle 8.11.1, AGP 9.1.0, Java 21

### Architecture Pattern
**Two-Layer Architecture** (simplified from Android's three-layer approach):
1. **UI Layer**: `feature/*` modules with Composables + ViewModels (MVVM)
2. **Data Layer**: `data/` module with Repositories + Data Sources

**Why no Domain layer?** Intentionally omitted to reduce complexity. Add it only when you have complex business logic or need to share logic between multiple ViewModels.

### State Management Pattern
All screens follow a consistent state pattern using `UiState<T>` wrapper (defined in `core:ui`):

```kotlin
// 1. Define screen data (immutable state)
data class HomeScreenData(
    val items: List<Item> = emptyList()
)

// 2. ViewModel with UiState
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val repository: HomeRepository
) : ViewModel() {
    private val _uiState = MutableStateFlow(UiState(HomeScreenData()))
    val uiState = _uiState.asStateFlow()

    // Sync state update
    fun updateItems(items: List<Item>) {
        _uiState.updateState { copy(items = items) }
    }

    // Async state update (auto handles loading/error)
    fun fetchItems() {
        _uiState.updateStateWith {
            repository.fetchItems()
        }
    }
}

// 3. Composable with StatefulComposable wrapper
@Composable
fun HomeRoute(
    viewModel: HomeViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    StatefulComposable(
        state = uiState,
        onRetry = { viewModel.fetchItems() }
    ) { data ->
        HomeScreen(data = data)
    }
}
```

**Key utilities** (in `core:ui/utils/`):
- `UiState<T>`: Wrapper with data, loading, error
- `updateState {}`: Synchronous state updates
- `updateStateWith {}`: Async operations with automatic loading/error handling
- `StatefulComposable`: Consistent loading/error UI
- `OneTimeEvent<T>`: Thread-safe one-time event consumption (navigation, snackbars)

### Navigation Pattern
Type-safe navigation using Kotlin serialization (no string-based routes):

```kotlin
// 1. Define route with @Serializable
@Serializable
data class ProfileRoute(val userId: String)

// 2. NavController extension for navigation
fun NavController.navigateToProfile(userId: String) {
    navigate(ProfileRoute(userId = userId))
}

// 3. NavGraph extension for screen registration
fun NavGraphBuilder.profileScreen(
    onShowSnackbar: suspend (String, SnackbarAction, Throwable?) -> Boolean
) {
    composable<ProfileRoute> {
        ProfileRoute(onShowSnackbar = onShowSnackbar)
    }
}

// 4. Extract params in ViewModel via SavedStateHandle
@HiltViewModel
class ProfileViewModel @Inject constructor(
    savedStateHandle: SavedStateHandle
) : ViewModel() {
    private val userId: String = savedStateHandle.toRoute<ProfileRoute>().userId
}
```

### Data Flow Pattern (Offline-First)
1. **UI observes repository Flow** (local database is single source of truth)
2. **Repository returns Flow from Room database**
3. **WorkManager syncs in background** (network, battery, storage constraints)
4. **Sync updates local database**
5. **UI automatically updates via Flow observation**

```kotlin
// Repository pattern example
class HomeRepositoryImpl @Inject constructor(
    private val localDataSource: LocalDataSource,
    private val networkDataSource: NetworkDataSource,
    @IoDispatcher private val ioDispatcher: CoroutineDispatcher
) : HomeRepository {

    // UI observes this Flow
    override fun observeData(): Flow<List<Data>> =
        localDataSource.observeData()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atick-faisal/Jetpack-Android-Starter](https://github.com/atick-faisal/Jetpack-Android-Starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
