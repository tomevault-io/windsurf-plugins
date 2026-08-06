---
trigger: always_on
description: Guidance for any contributor or AI agent working in **`photo_manager`**
---

# AGENTS.md

Guidance for any contributor or AI agent working in **`photo_manager`**
(`flutter_photo_manager`), including forks.

> **Portability rule for this file:** it is read by many people, across forks,
> using different tools and models. Keep it self‑contained and free of
> machine‑, user‑, or model‑specific assumptions (no hardcoded home paths, no
> "my setup" commands). Prefer environment variables and documented defaults so
> the guidance holds on any host.

## What this project is — and why caution matters

`photo_manager` is a Flutter plugin exposing album/asset (gallery) management
APIs across **Android, iOS, macOS, and OpenHarmony**. It sits very low in the
Flutter ecosystem: many widely‑used packages (image pickers, gallery viewers,
editors) depend on it transitively. A regression or a tightened constraint here
ripples out to a large number of downstream apps.

Treat every change as touching public, widely‑consumed API. The three standing
constraints, in priority order:

1. **Do not move the version floor.** Keep `environment: sdk: ">=2.13.0 <4.0.0"`
   and `flutter: ">=2.2.0"` in `pubspec.yaml` unchanged unless the maintainer
   explicitly asks. Do not use language/SDK features that raise the effective
   floor (e.g. `extension type` needs Dart 3.3, above the current floor).
2. **Preserve compatibility.** Prefer additive changes. Never remove or change
   the signature/semantics of an existing public API; deprecate instead.
3. **Keep native features semantically accurate and cross‑platform consistent.**
   A method must mean the same thing on every platform, return the same shape,
   and degrade predictably where a platform can't support it.

## Repository layout

| Path | What it is |
|------|-----------|
| `lib/photo_manager.dart` | Public barrel (`export`s). Add new public types here. |
| `lib/src/managers/` | `PhotoManager` (static entry), caching, notify managers. |
| `lib/src/types/` | `AssetEntity`, `AssetPathEntity` (`entity.dart`), `DarwinAsset`/`DarwinAssetPath` (`darwin.dart`), enums/types. |
| `lib/src/internal/` | `plugin.dart` (the channel layer), `editor.dart`, `constants.dart`, `enums.dart`. |
| `lib/src/filter/` | Classical + custom filter APIs. |
| `darwin/` | **Shared** iOS/macOS Objective‑C source. `ios/` and `macos/` are symlinks to `darwin/`. |
| `android/src/main/kotlin/com/fluttercandies/photo_manager/` | Android (Kotlin). Unit tests under `android/src/test/`. |
| `ohos/` | OpenHarmony (ArkTS `.ets`). |
| `example/` | Reference app; also where you add manual verification pages. |
| `test/` | Dart unit tests (`flutter test`). |

Darwin packaging is dual: `darwin/photo_manager.podspec` (CocoaPods) **and**
`darwin/photo_manager/Package.swift` (SPM). Keep both consistent when you add
source files, resources, or privacy manifest entries.

## Platform architecture — how to add platform-specific behavior

There is **one** `MethodChannel` (`com.fluttercandies/photo_manager`,
`PMConstants.channelPrefix`). `PMMethodChannel` auto‑injects a `cancelToken`.

Dart side: `PhotoManagerPlugin with BasePlugin, IosPlugin, AndroidPlugin,
OhosPlugin`. **Platform‑specific channel methods live in the matching mixin**
(`IosPlugin`, `AndroidPlugin`, `OhosPlugin`), each guarded with
`assert(Platform.isX)` (or a soft runtime guard returning an empty/neutral value
on unsupported platforms). They are reachable publicly via `PhotoManager.plugin.<method>`.

**Do not bloat `AssetEntity` / `AssetPathEntity` / `PhotoManager` with
platform‑specific members.** The codebase segregates platform APIs behind
namespaces; follow the established pattern that fits:

- **Mutations** → `PhotoManager.editor.darwin` / `.android` / `.ohos`
  (`Editor` in `editor.dart`), each guarded by a platform check that throws `OSError`.
- **Entity‑scoped reads** → `asset.darwin` / `path.darwin`, returning the
  lightweight `DarwinAsset` / `DarwinAssetPath` wrappers in `types/darwin.dart`.
  The getter performs the platform guard; the wrapper only forwards to `plugin`.
- **Library‑level / batch calls** → expose through `PhotoManager.plugin.<method>`
  (e.g. `getCloudIdentifiers`) rather than a bespoke static on `PhotoManager`.
- **Typed extra data** → nested types like `AlbumType.darwin` / `.ohos`.

When you add a channel method, wire all four sides: `PMConstants` string,
Dart mixin method, native handler, and (if user‑facing) the namespaced accessor.
Native dispatch: `PMPlugin.m` (`handleMethodResultHandler:`) for Darwin — mirror
an existing `else if` branch and reuse existing manager routines; Kotlin
`PhotoManagerPlugin.kt` for Android; `PhotoManagerPlugin.ets` for OHOS.

## Cross-platform semantics (non-negotiable)

- Decide the **contract first**: return type, units, null/empty behavior, and
  what happens on each unsupported platform — *then* implement per platform.
- Document per‑platform behavior in dartdoc using the existing bullet style:
  ```
  ///  * Android: ...
  ///  * iOS/macOS: ...
  ///  * OpenHarmony: ...
  ```
- Degrade predictably on unsupported platforms (empty map / `null` / `false` /
  empty list) instead of throwing — the deliberate exception is the `.darwin`
  accessor guard, which throws `OSError` by design.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluttercandies/flutter_photo_manager](https://github.com/fluttercandies/flutter_photo_manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
