---
trigger: always_on
description: Offline-first Android e-book reader: Jetpack Compose + Material 3, Clean Architecture + MVVM, Hilt DI, Room + FTS5. All data stays on-device — no accounts, no analytics, no sync. Supports EPUB / TXT / PDF / Markdown / FB2 / MOBI / JPG-PNG-WebP / ZIP / CBZ (read-only; DRM rejected). Docs and user-facing strings are Chinese; code, identifiers and comments are English.
---

# FlowReader (心流阅读)

Offline-first Android e-book reader: Jetpack Compose + Material 3, Clean Architecture + MVVM, Hilt DI, Room + FTS5. All data stays on-device — no accounts, no analytics, no sync. Supports EPUB / TXT / PDF / Markdown / FB2 / MOBI / JPG-PNG-WebP / ZIP / CBZ (read-only; DRM rejected). Docs and user-facing strings are Chinese; code, identifiers and comments are English.

Gradle modules: `:app`, `:core`, `:data`, `:domain`, `:feature:library`, `:feature:reader`. Package `com.flowreader.app`, applicationId `com.flowreader.app`, minSdk 26, compile/target SDK 35, GPL-3.0. Current build-file version is `56.4.1` (versionCode 5641), aligned with `CHANGELOG.md`'s newest entry v56.4.1.

## Commands

- `./gradlew assembleDebug` — development APK.
- `./gradlew assembleRelease` — R8 full-mode minify + resource shrink. **Deliberately signs with the debug key** (`signingConfig = signingConfigs.getByName("debug")` in `app/build.gradle.kts`) so CI can emit a testable `app-release.apk`.
- `./gradlew testDebugUnitTest` — all JVM unit tests (JUnit 4 + MockK + Robolectric; no emulator).
- `./gradlew :app:testDebugUnitTest --tests <FQCN>` — single test class (e.g. `com.flowreader.app.util.BookParserTest`).
- `./gradlew verifyKotlinStyle` — ktlint for every module **except `:app`** (root `build.gradle.kts` `subprojects` block) plus a lightweight whitespace gate over **all** `.kt`/`.kts` in the repo: any tab character or trailing whitespace fails the build. ktlint enforces a 140-char line cap and single-line signatures when they fit.
- `./gradlew recordRoborazziDebug` — records screenshot goldens under `app/src/test/snapshots/`; `./gradlew verifyRoborazziDebug` gates CI. Roborazzi 1.40 + Robolectric run on the JVM (`@GraphicsMode(NATIVE)`); capture API is `captureRoboImage(filePath = "src/test/snapshots/<name>.png") { content }` — `RoborazziRule` no longer exists in 1.40. Goldens currently: `library_shelf_light.png`, `library_skeleton_dark.png`.
- `./gradlew performanceBaseline` — builds debug+release, prints APK sizes vs `baseline/apk-size.properties` (currently debug 27482KB / release 10886KB), warns when an APK grew, then rewrites the baseline. Part of CI. Two traps: `PERF-WARNING` is a `println`, not a build failure, and the task **rewrites the baseline every run**, so a bogus number becomes the new reference. **Never judge a size regression from an incrementally-built APK.** AGP's incremental packaging (zipflinger) replaces a dex entry without rewriting the archive, leaving the old entry as a hole; after a dozen rebuilds in one session the debug APK carried **1114KB of inter-entry padding vs 73KB from a clean build** (a single 933KB hole ahead of `DebugProbesKt.bin`), reading as a 1063KB "growth" while the real delta was 8KB. Run `clean assembleDebug` first, or compare against a clean build in a throwaway `git worktree`.
- `./gradlew coverageSummary` — file-count test-breadth ratio (see Testing) with a 40% floor; currently **76.2% (48/63 files)**.
- `./gradlew clean` — when generated/KSP state looks stale.
- CI is `.github/workflows/ci.yml` (push/PR on `main`): `verifyKotlinStyle` → `testDebugUnitTest` → `coverageSummary` → `assembleDebug` → `verifyRoborazziDebug` → `performanceBaseline`. Do not claim a check ran unless you ran the Gradle task or inspected CI results.

## Toolchain

- JDK 17 and Android SDK 35 required.
- Gradle wrapper `9.6.1` downloaded from `mirrors.cloud.tencent.com`; wrapper/network failures are usually mirror-related.
- AGP `8.6.0`, Kotlin `2.1.0`, Compose BOM `2024.12.01`, Hilt `2.55`, Room `2.6.1` (KSP), ktlint `12.1.2`, Roborazzi `1.40.0`, Robolectric `4.14.1`, MockK `1.13.16`, coroutines `1.9.0`, Navigation Compose `2.8.5`, Coil `2.7.0`, JSoup `1.18.3`, Readium Kotlin Toolkit `3.1.2`, `desugar_jdk_libs 2.0.4`, `profileinstaller 1.4.1`.
- `coreLibraryDesugaring` enabled (`java.time` works on minSdk 26). Room schemas export to `data/schemas/` (the `:app` ksp arg for `app/schemas` is a leftover; the database lives in `:data` now). KSP output is under each module's `build/generated/ksp/`.
- `.editorconfig`: UTF-8, LF, 4-space indent, `max_line_length = 140`, ktlint `android_studio` style, `ktlint_function_naming_ignore_when_annotated_with = Composable`.

## Modules and dependency rules

Allowed direction: `feature:* → core/domain`, `data → core/domain`, `app → core/data/domain/feature:*`. Features must not depend on `:app`; `:data` must not depend on features; `:domain` must stay free of app/Room/Compose/Hilt.

- `:app` — composition root: Hilt graph, navigation shell, all screens/ViewModels, repository implementations, parsers (`util/`), widget, ContentProvider. Legacy home of most UI; screens are being migrated to `:feature:*` boundaries.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HuZaiGong/flowreader](https://github.com/HuZaiGong/flowreader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
