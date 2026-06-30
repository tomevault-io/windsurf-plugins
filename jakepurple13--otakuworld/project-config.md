---
trigger: always_on
description: provides `KmpRemoteSources` for in-app installation
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository.

## What this project is

OtakuWorld is a multi-app Android project with three main consumer apps: **MangaWorld** (manga
reader), **AnimeWorld** (anime streamer), and **NovelWorld** (novel reader). They share a large
common codebase and are actively being migrated to Kotlin Multiplatform (KMP) to eventually support
JVM/Desktop (via Compose Multiplatform). A Desktop build of MangaWorld (`mangaworld:desktop`)
already exists.

Apps contain no bundled sources — sources are loaded as external plugins/APKs at runtime via the
extension loader system.

## Build commands

```bash
# Build a specific app (use noFirebase flavor for local builds — no google-services.json needed)
./gradlew :mangaworld:assembleNoFirebaseDebug
./gradlew :animeworld:assembleNoFirebaseDebug
./gradlew :novelworld:assembleNoFirebaseDebug

# Build the desktop app
./gradlew :mangaworld:desktop:run

# Run all tests
./gradlew test

# Run tests for a specific module
./gradlew :UIViews:test
./gradlew :kmpuiviews:test

# Run a single test class
./gradlew :UIViews:test --tests "com.programmersbox.uiviews.ExampleUnitTest"

# Clean build
./gradlew clean
```

**Important:** Always use the `noFirebase` build variant for local development. This is set as the
default (`isDefault = true`). The `full` flavor requires `google-services.json` secrets.

## Build flavors and types

Three product flavors (dimension `version`):

- `noFirebase` — default for local dev, no Firebase dependency, appId suffix `.noFirebase`
- `noCloudFirebase` — Firebase crashlytics only, no cloud sync
- `full` — complete Firebase integration

Build types: `debug`, `release`, `beta` (beta = non-debuggable debug).

Flavor-specific Firebase utility implementations live in
`sharedutils/src/{noFirebase,noCloudFirebase,full}/java/`.

## Module structure

| Module                    | Purpose                                                                                                                         |
|---------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| `kmpmodels`               | KMP data models (`KmpItemModel`, `KmpInfoModel`, `KmpChapterModel`, `KmpStorage`, `KmpApiService`) — the source plugin contract |
| `kmpuiviews`              | KMP shared UI, ViewModels, repositories, DI modules, navigation (targets: Android, JVM, iOS)                                    |
| `UIViews`                 | Android-only shared UI layer extending `kmpuiviews`; `BaseMainActivity`, `GenericInfo` interface                                |
| `favoritesdatabase`       | KMP Room database for favorites, history, custom lists, recommendations                                                         |
| `datastore`               | KMP DataStore/protobuf settings handling                                                                                        |
| `datastore:mangasettings` | Manga-specific protobuf settings                                                                                                |
| `sharedutils`             | Firebase utilities with flavor-specific implementations                                                                         |
| `source_utilities`        | `NetworkHelper` for HTTP source plugins                                                                                         |
| `mangaworld`              | MangaWorld Android app, `GenericManga` implementation                                                                           |
| `mangaworld:shared`       | Shared manga reader UI (KMP)                                                                                                    |
| `mangaworld:desktop`      | JVM/Desktop Compose app for MangaWorld                                                                                          |
| `animeworld`              | AnimeWorld Android app                                                                                                          |
| `novelworld`              | NovelWorld Android app                                                                                                          |
| `novelworld:shared`       | Shared novel reader UI (KMP)                                                                                                    |
| `app`                     | OtakuWorld companion/manager app                                                                                                |

## Source plugin architecture

**This is the core extensibility pattern.** Sources are not bundled — they are loaded as external
plugins at runtime.

- `KmpApiService` (`kmpmodels`) — interface all sources implement. Key methods: `recent()`,
  `allList()`, `itemInfo()`, `chapterInfo()`, `search()`
- `KmpItemModel` / `KmpInfoModel` / `KmpChapterModel` — data model hierarchy flowing from source to
  UI
- `OtakuWorldCatalog` — fetches the remote extension index from `OtakuWorldSources` repo and
  provides `KmpRemoteSources` for in-app installation
- `KmpExternalApiServicesCatalog` / `KmpSources` — catalog abstraction for extension marketplace


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jakepurple13/OtakuWorld](https://github.com/jakepurple13/OtakuWorld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
