---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A Kotlin Multiplatform wrapper over the official Yandex MapKit SDK (Android `com.yandex.mapkit`,
iOS `YandexMapKit`/`YMK*` via CocoaPods), published as `ru.sulgik.mapkit:*`. It is not a Yandex
project.

The contract that drives every design decision: **common code sees a Kotlin-shaped MapKit, and native
types never leak into `commonMain`.** When a rule below looks arbitrary, check it against that
sentence — it usually explains it.

## Commands

```bash
./gradlew build                        # everything; iOS targets need CocoaPods installed
./gradlew build -PskipIosTarget=true   # Android-only, for machines without CocoaPods
```

Per-target compile checks (fastest feedback when changing `expect`/`actual` pairs — always run both):

```bash
./gradlew :yandex-mapkit-kmp:compileAndroidMain :yandex-mapkit-kmp:compileKotlinIosSimulatorArm64
```

```bash
./gradlew :yandex-mapkit-kmp-compose:compileAndroidMain :yandex-mapkit-kmp-compose:compileKotlinIosSimulatorArm64
```

Compiling for iOS only produces a klib; the Objective-C export runs when a framework is linked, and
it has its own failure modes. Anything that changes an exported signature should also be checked with:

```bash
./gradlew libraryLinkIosFramework
```

Tests live in `yandex-mapkit-kmp/src/{androidHostTest,androidDeviceTest,iosTest}` and in
`yandex-mapkit-kmp-compose/src/{commonTest,androidDeviceTest,iosTest}`; converter tests that need a
live MapKit object are device/simulator tests, the rest run on the JVM host:

```bash
./gradlew :yandex-mapkit-kmp-compose:allTests
```

```bash
./gradlew :yandex-mapkit-kmp-compose:testAndroidHostTest --tests "ru.sulgik.mapkit.ColorConvertionTest"
```

Formatting (Spotless with ktlint, configured in the root `build.gradle.kts`):

```bash
./gradlew spotlessApply
```

Public API of the four published modules is dumped into `<module>/api` and checked by
`libraryApiCheck`. There are two dumps per module: `<module>.klib.api` from the Kotlin ABI validator
(`commonMain` + `iosMain`) and `<module>.android.api` from the `dumpAndroidAbi` task in the root
`build.gradle.kts`, which runs `javap -public` over the Android main compilation because the
validator does not cover the target of `com.android.kotlin.multiplatform.library`:

```bash
./gradlew libraryApiDump   # after an intentional API change, run on macOS so iOS targets are included
```

KDoc site and docs:

```bash
./gradlew :dokkaGenerate    # renders into docs/kdoc, linked from mkdocs as "API docs"
```

```bash
mkdocs serve                # needs pip install mkdocs-material
```

Sample app — requires `MAPKIT_API_KEY=<key>` in `local.properties`, otherwise the buildKonfig step
fails with an explicit message. Android: `./gradlew :sample:composeApp:installDebug`. iOS: open
`sample/iosApp/iosApp.xcworkspace` in Xcode.

Style is enforced by `spotlessCheck`; correctness by `-Xexplicit-api=strict`, the compiler and
`libraryApiCheck`.

Within `1.x` the modules promise source compatibility, not binary compatibility — MapKit grows its
structures and enums, and the wrapper mirrors them as `data class`es and `enum class`es. README and
`docs/getting-started/contributing.md` state this; keep them in sync when the policy changes.

`main` is the only long-lived branch. Every pull request runs `.github/workflows/ci.yml` — `lint`,
`test-android`, `test-ios` and `api-check` in parallel, driven by the `library*` aggregate tasks
registered in the root `build.gradle.kts`. Instrumented tests run on an emulator only on pushes to
`main` and before a release, because Compose rendering tests need a real frame and are excluded from
the Android host run.

Releases are triggered manually (`.github/workflows/release.yml`, `workflow_dispatch` with a
version): CI verifies that `library_version`, the tag and the CHANGELOG section agree, runs the
emulator tests, builds and tests everything, publishes to Maven Central, and only then creates the
tag, the GitHub release and deploys the docs. `RELEASING.md` describes the procedure. Do not publish
locally.

## Architecture in one page

Four published modules plus a sample:

- `yandex-mapkit-kmp` — the wrapper itself, package `ru.sulgik.mapkit.*` mirroring MapKit's own
  packages (`geometry`, `map`, `location`, `logo`, `indoor`, `layers`, `user_location`, `mapview`).
  Users migrate by swapping the import prefix, so package and type names must keep matching MapKit.
- `yandex-mapkit-kmp-compose` — Compose Multiplatform rendering. Runs a **second composition** whose
  applier is `MapApplier` and whose nodes are map objects, so `@YandexMapComposable` content cannot
  contain UI composables.
- `yandex-mapkit-kmp-moko` / `-moko-compose` — moko-resources images as `ImageProvider`.

Every type falls into one of three shapes:

1. **Value type** — `data class` / `@JvmInline value class` in `commonMain`, converters per platform.
2. **Handle type** — `expect class` in common; `actual class X internal constructor(private val nativeX: NativeX)`
   with a member `toNative()` and a top-level `NativeX.toCommon()` in each platform source set.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SuLG-ik/yandex-mapkit-kmp](https://github.com/SuLG-ik/yandex-mapkit-kmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
