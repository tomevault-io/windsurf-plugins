---
trigger: always_on
description: Canonical project documentation for **AndroidAutoX**. This is the source of truth for both human contributors and AI coding agents working in this repository.
---

# AGENTS.md

Canonical project documentation for **AndroidAutoX**. This is the source of truth for both human contributors and AI coding agents working in this repository.

## Project overview

AndroidAutoX is a native Android app that applies "tweaks" to Google's Android Auto. It works by toggling Phenotype feature flags and Gearhead settings on a rooted device, exposing them through a simple button-per-tweak UI. Tweaks cover things like screen layout, appearance, video quality, and pre-activation of experimental Android Auto features.

- Application ID / package: `com.xiddoc.androidautox`
- App name: "AndroidAutoX"
- Repository: https://github.com/Xiddoc/AndroidAutoX
- Requires a **rooted** device.

## Tech stack

- Language: Java
- Build system: Gradle 8.7 with Android Gradle Plugin 8.5.2 (Gradle wrapper included); AndroidX
- minSdk 31 (Android 12), targetSdk 34, compileSdk 34 (Java 17 source/target)
- Root via **libsu** (`com.github.topjohnwu.libsu`): one persistent root shell for shell
  commands (`MainActivity.runSuWithCmd`), plus a **`RootService`** (`PhixitRootService`) that
  runs the platform `android.database.sqlite` API inside a root process to edit GMS's
  databases directly. No bundled `sqlite3` binary, and DB SQL is parameterized rather than
  built into shell strings.
- Release builds are minified/shrunk with R8 (`minifyEnabled` + `shrinkResources`); keep rules
  for libsu, the AIDL bridge, and reflection-based libs are in `app/proguard-rules.pro`.

## Repository layout

| Path | Description |
| --- | --- |
| `app/build.gradle` | Module build config: build types, signing, versioning, archive naming |
| `build.gradle`, `settings.gradle`, `gradle.properties` | Top-level Gradle config |
| `gradlew`, `gradlew.bat`, `gradle/` | Gradle wrapper |
| `app/src/main/AndroidManifest.xml` | App manifest / declared activities |
| `app/src/main/java/com/xiddoc/androidautox/` | All Java source |
| `app/src/main/java/com/xiddoc/androidautox/SplashActivity.java` | Entry point: disclaimer, root check, asset copy |
| `app/src/main/java/com/xiddoc/androidautox/MainActivity.java` | Main UI + all tweak handlers (~4300 lines) |
| `app/src/main/java/com/xiddoc/androidautox/AppsList.java` | App list helper |
| `app/src/main/java/com/xiddoc/androidautox/StreamLogs.java` | Log streaming view |
| `app/src/main/java/com/xiddoc/androidautox/AboutDialog.java` | About dialog |
| `app/src/main/java/com/xiddoc/androidautox/NoRootDialog.java` | Shown when root is unavailable |
| `app/src/main/java/com/xiddoc/androidautox/NotSuccessfulDialog.java` | Shown when a tweak fails |
| `app/src/main/java/com/xiddoc/androidautox/RebootDialog.java` | Prompts a reboot after a tweak |
| `app/src/main/java/com/xiddoc/androidautox/Utils/` | `UtilsLibrary`, `Version`, `BottomDialog` helpers |
| `app/src/main/res/layout/scrollview.xml` | Tweak buttons UI (the main tweak list) |
| `app/src/main/res/layout/activity_splash.xml` | Splash / disclaimer layout |
| `app/src/main/java/com/xiddoc/androidautox/AaxApp.java` | Application: configures the libsu root shell, inits `RootDb` |
| `app/src/main/java/com/xiddoc/androidautox/PhixitRootService.java` | libsu `RootService`: opens GMS DBs as root via the platform SQLite API |
| `app/src/main/java/com/xiddoc/androidautox/RootDb.java` | App-side handle that binds `PhixitRootService` and exposes `IPhixitRoot` |
| `app/src/main/aidl/com/xiddoc/androidautox/IPhixitRoot.aidl` | Root SQLite bridge interface (read/write partitions, query, exec) |
| `app/src/main/aidl/com/xiddoc/androidautox/Partition.aidl` + `Partition.java` | Parcelable {id, blob} crossing the binder |
| `app/src/main/res/values/strings.xml` | English source strings (localization source of truth) |
| `app/src/main/res/values-*/strings.xml` | ~33 translated locales (edited via PRs) |
| `app/src/main/java/com/xiddoc/androidautox/autox/` | AutoX virtual-display subsystem (pure-logic + framework glue) |
| `app/src/main/res/xml/automotive_app_desc.xml` | Car app descriptor (`<automotiveApp><uses name="template"/>`) |
| `docs/autox-architecture.md` | AutoX subsystem architecture: pipeline, class split, input-injection caveat |

## AutoX virtual-display subsystem

The `autox/` sub-package implements an isolated virtual-display projection pipeline
for the AutoX feature. Pure-logic classes (`AutoXDisplaySpec`, `CoordinateTranslator`,
`VirtualDisplayConfig`, `GestureSpec`, `AutoXTargetApp`, `AutoXAppRegistry`,
`AppLaunchPolicy`) have no Android imports and are covered at 100% by unit tests.
Framework-glue classes (`VirtualDisplayController`, `AppLauncher`, `GestureInjector` /
`LsposedInputInjector`, `AutoXCarAppService`, `AutoXSession`, `AutoXScreen`,
`AutoXForegroundService`) are excluded from the coverage gate. See
`docs/autox-architecture.md` for the full pipeline diagram and design rationale.

### Requirements: root + LSPosed (LSPosed-first single path)

AutoX requires **root** (baseline) **AND** the AndroidAutoX **LSPosed** module. This is a
deliberate single clean path — there is **no dual root-only-vs-LSPosed support**:

- **Trusted-display flag + cross-display input injection go through LSPosed ONLY.** Neither

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xiddoc/AndroidAutoX](https://github.com/Xiddoc/AndroidAutoX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
