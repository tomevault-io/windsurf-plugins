---
trigger: always_on
description: Guidance for AI assistants working on the **NovelLibrary** Android codebase.
---

# CLAUDE.md — NovelLibrary

Guidance for AI assistants working on the **NovelLibrary** Android codebase.

---

## Project Overview

NovelLibrary is an Android app (Kotlin) for reading web novels from multiple sources. It supports offline downloads, TTS playback, cloud sync via Firebase, a dynamic extension system for novel sources, and Cloudflare bypass. Current version: **1.2.0** (versionCode 123), min SDK 23, target SDK 36.

---

## Repository Layout

```
NovelLibrary/
├── app/src/main/java/io/github/gmathi/novellibrary/
│   ├── activity/          # Activities (NavDrawerActivity is the main entry point)
│   ├── fragment/          # Legacy XML-based fragments
│   ├── compose/           # Jetpack Compose UI (modern, preferred for new screens)
│   │   ├── search/        # PersistentSearchView component
│   │   ├── recentnovels/  # Recent novels tab
│   │   ├── cloudflare/    # Cloudflare resolver UI
│   │   ├── common/        # LoadingView, ErrorView, EmptyView, URLImage
│   │   ├── components/    # NovelItem, RecentlyUpdatedItem
│   │   └── theme/         # Material 3 theme
│   ├── viewmodel/         # MVVM ViewModels (SearchTermViewModel, RecentNovelsViewModel, etc.)
│   ├── database/          # SQLite helpers (DBHelper singleton, NovelHelper, WebPageHelper, etc.)
│   ├── model/             # Data models (Novel, WebPage, Genre, Download, …)
│   ├── network/           # Retrofit/OkHttp networking, Cloudflare bypass, source proxies
│   ├── service/           # Background services (TTS, Download, Firebase Messaging)
│   ├── worker/            # WorkManager jobs (BackupWorker, RestoreWorker)
│   ├── extension/         # Dynamic extension system (APK-based source plugins)
│   ├── cleaner/           # Source-specific HTML cleaners (HtmlCleaner base + overrides)
│   ├── domain/usecase/    # Business logic use cases
│   ├── util/              # Constants, Utils, DataCenter (SharedPrefs), Logs, notifications
│   ├── AppModule.kt       # Injekt DI bindings
│   └── NovelLibraryApplication.kt  # App init: DI, SSL, Firebase, notifications
├── app/src/test/          # Unit tests (minimal — mostly example tests)
├── app/src/androidTest/   # Instrumented tests (minimal — mostly example tests)
├── app/src/mirror/        # Mirror flavor resources
├── docs/                  # Technical documentation (architecture, migration guides, etc.)
├── gradle/
│   └── libs.versions.toml # Version catalog — all dependency versions defined here
├── app/build.gradle       # App module Gradle config
├── build.gradle           # Root Gradle config
├── settings.gradle        # Module declarations
└── gradle.properties      # Build flags (parallel builds, config cache, R8, etc.)
```

---

## Build System

**Toolchain:** Gradle + Android Gradle Plugin (AGP) 9.1.0, Kotlin 2.3.10, KSP 2.3.6, JVM 17.

All dependency versions are centralized in `gradle/libs.versions.toml`. When adding or updating a dependency, edit the version catalog first, then reference `libs.*` aliases in `app/build.gradle`.

### Build Variants

| Flavor   | Description                                                        |
|----------|--------------------------------------------------------------------|
| `normal` | Default production build (applicationId: `io.github.gmathi.novellibrary`) |
| `mirror` | Alternative build with `.mirror` suffix — can coexist on the same device |
| `canary` | Debug build with LeakCanary enabled for memory leak detection       |

### Common Gradle Commands

```bash
# Debug build (normal flavor — the default)
./gradlew assembleNormalDebug

# Release build
./gradlew assembleNormalRelease

# Run unit tests
./gradlew test

# Run instrumented tests (requires connected device/emulator)
./gradlew connectedAndroidTest

# Lint
./gradlew lint

# Canary (memory leak detection) debug build
./gradlew assembleCanaryDebug
```

### Required Config Files (not in repo)

| File | Location | Purpose |
|------|----------|---------|
| `google-services.json` | `app/` | Firebase configuration |
| `keystore.properties` | project root | Release signing config |

Without `google-services.json` the build will fail. Download from the project's Google Drive (see `README.md`).

---

## Architecture

The app follows **MVVM** with a Repository pattern and is partway through a migration from legacy XML layouts to **Jetpack Compose**.

```
Presentation  →  ViewModel  →  Repository / UseCase  →  Database / Network
(Activity/Fragment/Compose)     (domain/usecase/)        (database/ network/)
```

### Dependency Injection

Uses **Injekt** (not Hilt/Dagger). Bindings are registered in `AppModule.kt` and resolved via `instance<T>()` or `injectLazy<T>()`. Do not introduce Hilt unless migrating the entire DI layer.

### UI: Compose vs. Legacy Views

- **New screens** should be written in Jetpack Compose using Material 3.
- **Legacy screens** use XML layouts with Data Binding / View Binding — leave them as-is unless actively migrating.
- Compose screens live under `compose/` and are hosted in Activities or `ComposeView`-based Fragments.
- The `PersistentSearchView` component (`compose/search/`) is the canonical search UI. See `docs/compose-search-quick-start.md` and `docs/compose-search-migration-guide.md`.

### Reactive Programming


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gmathi/NovelLibrary](https://github.com/gmathi/NovelLibrary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
