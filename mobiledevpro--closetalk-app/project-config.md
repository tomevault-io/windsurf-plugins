---
trigger: always_on
description: This is an Android chat app built with **Jetpack Compose**, **Kotlin 2.4**, and **MVI (
---

# AGENTS.md - AI Development Guide for Android Jetpack Compose MVI Template

## Project Overview

This is an Android chat app built with **Jetpack Compose**, **Kotlin 2.4**, and **MVI (
Model-View-Intent) architecture** with heavy modularization. The codebase separates concerns into 8
core modules and 8+ feature modules, each with clear responsibilities.

**Key Stack**: Compose, Kotlin Coroutines, Koin DI, Room DB, Firebase, Material 3, Navigation
Compose

---

## Architecture Patterns

### MVI (Model-View-Intent)

**Pattern**: State-driven UI with unidirectional data flow.

- **ViewModel**: Extends `BaseViewModel<State>` from `core:ui` module
- **State**: Sealed interface implementing `UIState` (e.g., `Loading`, `Success(data)`,
  `Fail(error)`)
- **Intent**: User actions trigger state changes via UseCase → ViewModel → StateFlow

**Example in `feature:people_list`**:

```kotlin
class PeopleListViewModel(val getPeopleListUseCase: GetPeopleListUseCase) : BaseViewModel<PeopleProfileUIState>() {
    override val initialState = PeopleProfileUIState.Loading
    override fun observeState() = getPeopleListUseCase.execute()
        .map { result -> try { PeopleProfileUIState.Success(result.getOrThrow()) } catch (t) { PeopleProfileUIState.Fail(t) } }
}

sealed interface PeopleProfileUIState : UIState {
    data object Loading : PeopleProfileUIState
    data class Success(val profileList: List<PeopleProfile>) : PeopleProfileUIState
    data class Fail(val throwable: Throwable) : PeopleProfileUIState
}
```

**Consume in Compose**:

```kotlin
@Composable
fun PeopleListScreen(vm: PeopleListViewModel = rememberNavViewModel { modules }) {
    val state by vm.uiState.collectAsState()
    when(state) { /* render based on state */ }
}
```

### Modular Architecture

**Two layers**:

- **`:core:*` modules**: Shared infrastructure (UI, DI, Navigation, Domain, Coroutines, Analytics,
  Database, Utils)
- **`:feature:*` modules**: Feature screens (people_list, home, chat_list, user_profile, etc.)

**Dependency flow**: `Feature modules → Core modules` (never the reverse). Navigation orchestrates
feature composition.

**Module names use underscores**: `people_list` → namespace `com.mobiledevpro.people.list` (via
`core-module.gradle.kts`)

---

## Build System & Conventions

### Gradle Plugins

Located in `build-logic/src/main/kotlin/`:

- **`core-module.gradle.kts`**: Applied to all core modules. Sets namespace, minSdk, SDK versions,
  desugaring, flavors (dev/production)
- **`feature-module.gradle.kts`**: Extends core-module, adds Compose, auto-includes core
  dependencies (ui, di, domain, coroutines, util, analytics, lifecycle, coil)
- **`core-compose-module.gradle.kts`**: For core modules needing Compose (e.g., `core:ui`)
- **`kotlin-convention.gradle.kts`**: Kotlin/JDK configuration

**Example**: A feature module only needs `plugins { id("feature-module") }` to get all standard
dependencies.

### Version Management

- **`gradle/libs.versions.toml`**: Single source for all versions (Kotlin 2.4, Compose BOM
  2026.05.01, AGP 9.2.1, Koin 4.2.1, Room 2.8.4, Firebase BOM 34.14.0)
- Release versions in `gradle/libs.versions.toml` (`app-version-code`, `app-version-name`)

### Build Variants

- **Flavors**: `dev` (default, app ID `.apptemplate.compose`) and `production` (`.closetalk.app`)
- **Build Types**: Debug (Crashlytics disabled) and Release (minified, Crashlytics enabled)
- **Build output**: `.aab` files auto-renamed via `RenameBundleTask` in `app/build.gradle.kts`

### Key Build Commands

```bash
./gradlew clean build --profile          # Build with performance profiling
./gradlew bundleProductionRelease        # Create AAB for Play Store
./gradlew createModuleGraph              # Update module graph in README
./gradlew --profile                      # Generate build metrics
```

---

## Dependency Injection with Koin

### Pattern

- **Module registration**: Each feature defines a `di/Module.kt` with
  `val featureNameModule = module { ... }`
- **Scope pattern**: Use `scope<ViewModel>` to tie dependencies to ViewModel lifecycle
- **ViewModels registered as**: `viewModelOf(::ClassName)` (uses SavedStateHandle automatically)
- **Other dependencies**: `scopedOf(::UseCase)` for classes tied to scope, `singleOf()` for
  singletons

**Example from `feature:people_list`**:

```kotlin
val featurePeopleListModule = module {
    scope<PeopleListViewModel> {
        viewModelOf(::PeopleListViewModel)
        scopedOf(::GetPeopleListUseCase)
    }
}
```

### Consuming in Compose

Via `core:di` helper: `rememberNavViewModel<ViewModelType> { listOf(module) }` - handles module
loading/unloading automatically.

---

## State & Coroutines

### StateFlow Pattern (core:ui BaseViewModel)

- `uiState: StateFlow<State>` exposed lazily
- Uses `SharingStarted.WhileSubscribed(stopTimeoutMillis=0, replayExpirationMillis=9000ms)`
- Handles lifecycle safety automatically

### Testing Coroutines

- Use **Turbine** (`app.cash.turbine`) for Flow testing: `vm.uiState.test { awaitItem() ... }`
- Use **Robolectric** for instrumented tests without device
- Use `StandardTestDispatcher` with test scheduler for deterministic timing
- See `PeopleListViewModelTest.kt` for pattern

---

## Testing Conventions

### Test Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mobiledevpro/closetalk.app](https://github.com/mobiledevpro/closetalk.app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
