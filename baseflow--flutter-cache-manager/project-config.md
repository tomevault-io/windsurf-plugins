---
trigger: always_on
description: Technical reference for AI agents and contributors developing in this repository.
---

# flutter_cache_manager agent instructions

Technical reference for AI agents and contributors developing in this repository.

Process and conduct live in their own files: contribution workflow in
[CONTRIBUTING.md](CONTRIBUTING.md), the [Contributor Covenant Code of
Conduct](CODE_OF_CONDUCT.md) (report unacceptable behavior to
[hello@baseflow.com](mailto:hello@baseflow.com)).

## Scope and stack

- This repo is the **Flutter Cache Manager** monorepo maintained by [Baseflow](https://baseflow.com).
- It contains two Dart packages (not a federated plugin). Work inside the **specific package** you are changing; there is no Melos or root pub workspace, and neither should be added unless the team decides to.
- Run Flutter and Dart commands with the same tooling CI uses (`flutter`, `dart`). Nothing in this repo requires anything else. If you happen to manage SDK versions locally with [fvm](https://fvm.app), prefix commands with `fvm`; that is a personal setup choice and is never checked in.

### Prerequisites

- Basic Dart and Flutter knowledge
- A working Flutter SDK installation (stable channel, matching CI — currently Flutter **3.47.4** in workflows)
- Comfort with filesystem / HTTP caching concepts helps, but is not required to start
- For running or building the example on iOS/macOS, access to a Mac is required
- Android example builds require JDK 17

### Reference documentation

This package is primarily a **Dart/Flutter library** (file download + disk cache), not a federated platform plugin. Prefer official Flutter/Dart docs and this repo's existing code for hands-on work:

- [Using packages](https://docs.flutter.dev/packages-and-plugins/using-packages)
- [Developing packages & plugins](https://docs.flutter.dev/packages-and-plugins/developing-packages)
- [Effective Dart](https://dart.dev/effective-dart)
- [pub versioning philosophy](https://dart.dev/tools/pub/versioning)

## Packages in this repo

| Package | Role |
|---------|------|
| `flutter_cache_manager` | Core cache manager: download, store, and serve files with configurable TTL / capacity |
| `flutter_cache_manager_firebase` | Optional `FileService` / cache manager integration for `firebase_storage` (`gs://` → HTTPS) |

There is no Melos workspace. Each package has its own `pubspec.yaml`, tests, and CI workflow.

## Architecture overview

```
App → CacheManager / DefaultCacheManager / custom Config
    → CacheStore (mem cache + CacheInfoRepository)
    → WebHelper + FileService (HTTP or custom, e.g. Firebase)
    → FileSystem (IO or memory on web)
```

Platform defaults for `CacheInfoRepository` (see `lib/src/config/_config_io.dart`):

- **Android / iOS / macOS** → `CacheObjectProvider` (sqflite)
- **Windows / Linux** → `JsonCacheInfoRepository`
- **Web** → non-storing provider

Custom `Config` can override `repo`, `fileSystem`, and `fileService`. Prefer extending or composing these abstractions rather than forking `CacheManager` internals.

**Persistence**: treat durability seriously — especially `JsonCacheInfoRepository` (full-file rewrite). Prefer serialized writes and atomic replace (temp + rename) over long debounce windows that can lose data on process kill. Do not reintroduce long debounce delays for JSON persistence without an explicit durability strategy (flush on close is not enough for force-stop).

## Authoritative project structure

- Root overview: `README.md` (symlink to `flutter_cache_manager/README.md`)
- Contribution workflow: `CONTRIBUTING.md`
- App-facing API: `flutter_cache_manager/lib/flutter_cache_manager.dart`
- Core manager: `flutter_cache_manager/lib/src/cache_manager.dart`
- Default / image managers: `flutter_cache_manager/lib/src/cache_managers/`
- Config (IO / web / unsupported): `flutter_cache_manager/lib/src/config/`
- In-memory + DB orchestration: `flutter_cache_manager/lib/src/cache_store.dart`
- Download / HTTP: `flutter_cache_manager/lib/src/web/`
- Cache metadata storage: `flutter_cache_manager/lib/src/storage/cache_info_repositories/`
  - `CacheObjectProvider` — sqflite (default on Android / iOS / macOS)
  - `JsonCacheInfoRepository` — JSON file (default on Windows / Linux)
  - `NonStoringObjectProvider` — web / no persistence
- File system abstraction: `flutter_cache_manager/lib/src/storage/file_system/`
- Firebase package: `flutter_cache_manager_firebase/lib/`
- Example app: `flutter_cache_manager/example/`
- CI:
  - `.github/workflows/build.yaml` — `flutter_cache_manager`
  - `.github/workflows/build-firebase.yaml` — `flutter_cache_manager_firebase`

## Where to make changes

- **Public API / docs for app developers** → `flutter_cache_manager/lib/` exports and `README.md`
- **Download / HTTP behavior** → `lib/src/web/`
- **Persistence / metadata** → `lib/src/storage/cache_info_repositories/`
- **Cache eviction / mem cache** → `lib/src/cache_store.dart`
- **Firebase integration** → `flutter_cache_manager_firebase/` only
- **Never** put Firebase-specific logic in `flutter_cache_manager`, or platform-specific logic in the core package when it belongs in config hooks or the Firebase package

Keep changes minimal in scope — one concern per change; match existing naming, error-handling (`FlutterError.reportError` where used), and testing patterns.

## Development setup


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Baseflow/flutter_cache_manager](https://github.com/Baseflow/flutter_cache_manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
