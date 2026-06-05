---
trigger: always_on
description: - **Requires** `TMDB_API_KEY=<token>` in `local.properties` (root). Missing key = empty BuildConfig field, not a build error.
---

# CLAUDE.md — Android Template Project

## Build & Run

- **Requires** `TMDB_API_KEY=<token>` in `local.properties` (root). Missing key = empty BuildConfig field, not a build error.
- Build/run via Android Studio or `./gradlew assembleDebug`.
- Run all tests: `./gradlew test`
- Run Konsist architecture tests: `./gradlew :test:konsist:test`
- Run Spotless check: `./gradlew spotlessCheck` — fix: `./gradlew spotlessApply`

---

## Module Structure

All modules are declared in `settings.gradle.kts`. Key groupings:

| Path | Role |
|---|---|
| `:app` | Entry point; depends on all feature + core modules |
| `:feature:movie-catalog` | Feature module (UI + ViewModel only) |
| `:core:data` | Repository implementations (single source of truth) |
| `:core:domain` | Use case interfaces/impls + repository interfaces |
| `:core:model` | Domain/business models only |
| `:core:network:api/retrofit/noop` | Network abstraction + Retrofit impl |
| `:core:database:api/room/noop` | Database abstraction + Room impl |
| `:core:presentation:mvi/theme/common-ui/viewmodel` | Shared UI infra |
| `:core:navigation:api/navigation-compose` | Navigation abstraction |
| `:core:bridge-di` | Prevents cyclic Gradle DI dependencies |
| `:architecture:mapper` | Mapper interfaces only |
| `:utils:koin`, `:utils:random` | Shared utilities |
| `:test:konsist` | Architecture rule tests (Konsist) |
| `:test:doubles:*` | Test doubles for DB, network, models |

**Rule**: Feature modules depend only on core modules, never on each other.  
**Rule**: Core modules never depend on feature or app modules.  
**Rule**: Only put classes in `:core` if used by 2+ feature modules.

---

## Convention Plugins (build-logic)

Every module uses one of these in its `build.gradle.kts` — never configure Android/Kotlin directly:

| Plugin ID | Use for |
|---|---|
| `com.pantelisstampoulis.library.core` | Any Android library module |
| `com.pantelisstampoulis.library.feature` | Feature modules (adds Compose, Serialization, domain/presentation deps) |
| `com.pantelisstampoulis.application.core` | `:app` module |
| `com.pantelisstampoulis.application.compose` | `:app` Compose setup |
| `com.pantelisstampoulis.koin` | Adds Koin BOM + common deps |
| `com.pantelisstampoulis.room` | Adds Room + KSP |
| `com.pantelisstampoulis.testing` | Adds test doubles, mockative, turbine, truth |
| `com.pantelisstampoulis.konsist` | Konsist architecture tests |
| `com.pantelisstampoulis.compose` | Compose compiler + dependencies |
| `com.pantelisstampoulis.kotlin.serialization` | KotlinX Serialization |

Use `namespaceWithProjectPackage(suffix = "foo.bar")` in `android { namespace }` — do not hardcode the full package.

SDK/Java version config comes from `gradle.properties` (`configuration.android.*`). Do not hardcode these values.

---

## Architecture: Clean Architecture + MVI

### Layer dependencies (enforced by Konsist)
```
network ──┐
database ──┤──► data ──► domain ──► model
           │              │
           └──────────────┤
                          ▼
                  feature (UI + ViewModel)
                       │
                  presentation
```

### MVI pattern
- ViewModels extend `MviViewModel<Event, UiState, SideEffect>` from `:core:presentation:mvi`
- `Event` = user intent (sealed interface, one per screen)
- `UiState` = immutable data class implementing `UiState` interface
- `SideEffect` = one-time effects (navigation, toasts) via `Channel`
- ViewModels call `setState { copy(...) }`, `setEffect { ... }`, never expose mutable state
- UI collects `viewState` as `StateFlow`, `effect` as `Flow`

### Use Cases
- Interface + `internal` impl pattern: `interface FooUseCase`, `internal class FooUseCaseImpl`
- Must reside in `..domain..usecase..` package
- Single public method: `override operator fun invoke(input: T): Flow<ResultState<R>>`
- Impl class uses `onStartCatch()` extension for coroutine error handling
- ViewModels **must not** receive repositories directly — use cases only (Konsist-enforced)

### Repository
- Interface in `:core:domain` at `..domain..repository` package
- Implementation (`*RepositoryImpl`) in `:core:data` at `..data..repository` package
- Every `RepositoryImpl` must have a corresponding `RepositoryImplTest` (Konsist-enforced)
- Returns `Flow<ResultState<T>>` — never suspends directly

### ResultState
```kotlin
// onLoading { }, onSuccess { data -> }, onError { error -> }
```

### Mappers
All mappers implement typed interfaces from `:architecture:mapper`:
- `ApiToDomainMapper<ApiModel, DomainModel>` → `fromApiToDomain()`
- `DbToDomainMapper<DbModel, DomainModel>` → `fromDbToDomain()`
- `ApiToDbMapper<ApiModel, DbModel>` → `fromApiToDb()`
- `DomainToUiMapper<DomainModel, UiModel>` → `fromDomainToUi()`

---

## Naming Conventions (Konsist-enforced)

| Class type | Suffix | Package |
|---|---|---|
| Use case interface | `UseCase` | `..domain..usecase..` |
| Use case impl | `UseCaseImpl` | `..domain..usecase..` |
| Repository interface | `Repository` | `..domain..repository` |
| Repository impl | `RepositoryImpl` | `..data..repository` |
| Network models | `ApiModel` | `..network.model` |
| Database models | `DbModel` or `Entity` | `..database.model` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pantstamp/android-template-project](https://github.com/pantstamp/android-template-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
