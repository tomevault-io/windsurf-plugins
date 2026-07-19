---
trigger: always_on
description: Mellow is a native Android music player for Jellyfin. Kotlin, Jetpack Compose, Media3, offline-first Room database.
---

# AGENTS.md — Agentic Coding Guidelines for Mellow

## Project Overview

Mellow is a native Android music player for Jellyfin. Kotlin, Jetpack Compose, Media3, offline-first Room database.

### Code Structure

```
mellow/
├── app/                    # Application entry, DI wiring, navigation
├── core/
│   ├── common/             # Shared utilities, result types, extensions
│   ├── model/              # Domain models (pure Kotlin, no Android deps)
│   ├── network/            # Jellyfin SDK wrapper (jellyfin-sdk-kotlin)
│   ├── database/           # Room DB, DAOs, entities, migrations
│   ├── data/               # Repository implementations (offline-first bridge)
│   └── player/             # Media3 player, MediaLibraryService, Android Auto
├── feature/
│   ├── home/               # Home screen (recent, favorites, quick access)
│   ├── library/            # Library browser (albums, artists, genres)
│   ├── player/             # Now playing, queue, lyrics
│   ├── search/             # Global search
│   └── settings/           # App configuration
└── sync/                   # WorkManager-based library sync
```

### Module Dependency Rules (HARD BLOCKS)

- `core/model` has ZERO Android dependencies (pure Kotlin data classes)
- `core/network` NEVER imports Room classes
- `core/database` NEVER imports jellyfin-sdk classes
- `core/data` is the ONLY module that bridges network ↔ database
- `feature/*` modules NEVER depend on each other
- Only `app` may depend on all modules (wiring)

### Offline-First Architecture (CRITICAL)

Room is the source of truth. The Jellyfin API is a sync target.

- Read path: Always read from Room. Background sync keeps it fresh.
- Write path: Write to Room immediately. Sync to server when online.
- Favorites, ratings, play counts all work offline and sync later.

**Never bypass Room to read directly from the API in feature code.**

## Build Commands

```bash
# Build debug APK
./gradlew assembleDebug

# Build release APK
./gradlew assembleRelease

# Run all unit tests
./gradlew test

# Run specific module tests
./gradlew :core:database:test
./gradlew :core:player:test

# Run Android instrumented tests
./gradlew connectedAndroidTest

# Lint check
./gradlew lint

# Check all (build + test + lint)
./gradlew check
```

### Gradle Wrapper

If the wrapper is missing, generate it:
```bash
gradle wrapper --gradle-version 8.12
```

## Code Style Guidelines

### Kotlin

1. **Error Handling**:
   - Use `MellowResult<T>` (sealed interface in `core/common`) for all repository methods
   - Never throw exceptions from repositories — wrap in `MellowResult.Error`
   - ViewModels convert `MellowResult` to UI state

2. **Imports**:
   - Group: stdlib, kotlinx, android/androidx, third-party, project
   - Use explicit imports (no wildcard `*`)

3. **Naming**:
   - `camelCase` for functions, properties, local variables
   - `PascalCase` for classes, interfaces, type aliases, composables
   - `SCREAMING_SNAKE_CASE` for constants
   - Prefix unused variables with `_`

4. **Constructors and IO**:
   - NEVER perform IO in constructors or `init {}` blocks
   - Use suspend functions for all IO operations

5. **Coroutines**:
   - Use `Flow<T>` for observable data streams from Room
   - Use `StateFlow<T>` in ViewModels for UI state
   - Use `viewModelScope` for coroutine launches in ViewModels
   - Never use `GlobalScope`

6. **Formatting**:
   - Follow standard Kotlin style (ktlint defaults)
   - Max line length: 120 characters

### Jetpack Compose

1. **Composable Functions**:
   - `@Composable` functions that emit UI start with uppercase (e.g., `AlbumCard`)
   - Accept `Modifier` as first optional parameter: `fun AlbumCard(modifier: Modifier = Modifier, ...)`
   - Use `remember` and `derivedStateOf` for computed values

2. **State Management**:
   - ViewModels expose `StateFlow<UiState>`
   - Screens collect state with `collectAsStateWithLifecycle()`
   - One-shot events use `Channel<Event>` consumed via `LaunchedEffect`

3. **Lists**:
   - Use `LazyColumn`/`LazyVerticalGrid` for all scrollable lists
   - Use Paging 3 with `collectAsLazyPagingItems()` for large datasets
   - Always provide stable `key` parameter

4. **Images**:
   - Use Coil `AsyncImage` for all network images
   - Provide `placeholder` (blurhash) and `error` drawables
   - Set explicit `contentScale` and `contentDescription`

5. **Animations**:
   - Use `Crossfade` or `AnimatedContent` for icon/content swaps — they skip animation on first composition automatically
   - Use `animateFloatAsState` / `animateDpAsState` for state-driven value changes — also skips initial composition
   - Use `Animatable` only for imperative fire-and-forget effects (ripples, press feedback) triggered by user taps
   - **Never use `LaunchedEffect(stateKey)` + `Animatable.animateTo()`** for state-driven animations — it animates on first composition. Use `Crossfade`/`animateXAsState` instead
   - Spring easing for pop/overshoot: `spring(dampingRatio = 0.45f, stiffness = 400f)`
   - All animated icons live in `core/designsystem/component/`: `AnimatedPlayPause.kt`, `AnimatedHeartIcon.kt`, `AnimatedDownloadIcon.kt`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Malinskiy/mellow](https://github.com/Malinskiy/mellow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
