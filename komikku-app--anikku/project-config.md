---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

**Anikku** is an Android anime/movie watcher app (min SDK 26, target SDK 36, JVM 17 / Kotlin). It's based on Aniyomi with additional features from Mihon, TachiyomiSY, and Komikku (manga fork). The app includes:
- Anime discovery and playback with configurable mpv-android player
- Local downloading and offline viewing
- Multiple tracker support (MyAnimeList, AniList, Kitsu, Simkl, Shikimori, Bangumi)
- Anime recommendations and metadata editing
- Library management with categories, tagging, and filtering
- Multi-source browsing and feed tabs

**Application ID**: `app.anikku` (debug variant: `app.anikku.dev`)

---

## Build & Development Environment

**Tech Stack**: Jetpack Compose + Material3 • Voyager navigation • SQLDelight database • Injekt DI

**JDK/Gradle**: JVM 17 (compiles to Kotlin JVM 17 target) • Gradle 9.3+ • Android SDK 36 (compileSdk)

### Essential Build Commands

```bash
# Code formatting (required before committing)
./gradlew spotlessApply           # Auto-fix formatting
./gradlew spotlessCheck           # Verify formatting (runs in CI)

# Build variants
./gradlew :app:compileDebugKotlin # Compile-only check (faster than assembleDebug)
./gradlew assembleDebug           # Debug APK (local testing)
./gradlew assemblePreview         # Preview (beta) APK (CI equivalent; use for consistency)
./gradlew assembleRelease         # Release APK

# Testing
./gradlew testReleaseUnitTest # Unit tests for release build
./gradlew test                # All unit tests

# Full CI build with telemetry and updater
./gradlew assemblePreview -Pinclude-telemetry -Penable-updater

# Install to device/emulator
./gradlew installDebug

# SQLDelight database regeneration (after .sq or .sqm schema changes)
./gradlew :data:generateSqlDelightInterface

# Clean build daemon (if OOM occurs)
./gradlew --stop
```

**Build types**: `debug`, `release`, `releaseTest`, `foss`, `preview` (default), `benchmark`

**Gradle `-P` flags**:

| Flag | Effect                                   |
|------|------------------------------------------|
| `include-telemetry` | Include Firebase Analytics + Crashlytics |
| `enable-updater` | Enable In-app update checker             |
| `disable-code-shrink` | Skip ProGuard/R8 minification            |
| `include-dependency-info` | Dependency metadata in APK               |

## Code Marking Convention

All Anikku-specific additions or modifications **must** be surrounded with comment markers:

```kotlin
// ANK -->
// your code here
// ANK <--
```

This is how Anikku-specific changes are tracked relative to the upstream Aniyomi fork. Always apply these markers to any changes you make.

All fork markers used in this codebase:

| Marker | Origin | Notes |
|--------|--------|-------|
| `// ANK` | Anikku-specific | Use for all new Anikku code |
| `// SY` | TachiyomiSY | Keep intact during upstream merges |
| `// KMK` | Komikku | Keep intact during upstream merges |
| `// AY` | Aniyomi | Keep intact during upstream merges |
| `// EXH` | E-Hentai (legacy) | Avoid; use `// ANK` for new code |

---

## Architecture

Multi-module layered architecture (22 modules) with MVVM + MVI patterns.

### Module Layout

| Module | Purpose |
|--------|---------|
| `app/` | UI (Voyager Screens + Compose), DI, workers, build variants |
| `domain/` | Use cases, models, repository interfaces |
| `data/` | SQLDelight database, repository implementations |
| `core:common/` | Network (OkHttp), security, storage, shared utils |
| `core:archive/` | Archive reading utilities |
| `core-metadata/` | Comic-info metadata parsing |
| `source-api/` | Extension `Source` interface + local source |
| `presentation-core/` | Shared Compose components |
| `i18n-ank/` | **Anikku-specific strings** → `AMR` resource class |
| `i18n/` | Mihon strings → `MR` (frozen upstream) |
| `i18n-sy/` | TachiyomiSY strings → `SYMR` (frozen upstream) |
| `i18n-aniyomi/` | Aniyomi strings → `AYMR` (frozen upstream) |
| `i18n-kmk/` | Komikku strings → `KMR` (frozen upstream) |
| `i18n-animiru/` | Animiru strings → `AMMR` (frozen upstream) |
| `presentation-widget/` | Home-screen Glance widget |
| `flagkit/` | Country-flag drawables |
| `telemetry/` | Firebase/Crashlytics (noop unless `-Pinclude-telemetry`) |
| `buildSrc/` | Custom Gradle plugins and build logic |

**Dependency flow**: `app` → `domain` → `source-api`; `data` implements `domain` repos.

**Data flow:** `source-api` extensions → `data` repositories → `domain` use cases → `presentation-core` ViewModels → `app` screens

**Navigation:** Voyager screens/tabs. Screen classes live in `app/src/main/java/.../ui/`.

**DI:** Injekt (lightweight custom container). Dependencies are registered at app startup and retrieved via `Injekt.get<T>()` or by implementing `Injekt.inject<T>()` delegation.

**Database:** SQLDelight with migrations in `data/src/main/sqldelight/`. Schema changes require a new migration file.

**Concurrency:** Kotlin coroutines + Flow for new code. RxJava remains in the `source-api` layer (extension compatibility).

## Key Technologies

- **UI:** Jetpack Compose + Material 3, Coil 3 for image loading

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [komikku-app/anikku](https://github.com/komikku-app/anikku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
