---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Vadhod APK Extractor** — a 100% offline, privacy-first Android app (package `com.vadhod.apkextractor`)
that lists installed apps (System / User tabs) and extracts their APKs to a user-picked folder. Simple
apps export as a single `.apk`; split apps bundle base + splits into a reinstallable `.apks` zip. The
visual identity — the Anthropic/Claude "soft pastel" palette — is treated as a headline feature, not
decoration.

Single Gradle module (`:app`), single Activity, Jetpack Compose + Material 3, Kotlin 2.4 (K2), MVVM + UDF.

## Build & test

Use the Gradle wrapper. On Windows use `gradlew.bat` (or `./gradlew` from the Bash tool).

```bash
./gradlew assembleDebug          # debug APK — must stay green between tasks (rules.md §E-26)
./gradlew testDebugUnitTest      # JVM unit tests
./gradlew assembleRelease        # R8/minify + resource shrink; verify release path stays clean
./gradlew connectedDebugAndroidTest   # instrumented/Compose tests (needs a device)
./gradlew installDebug           # push to connected device
```

Run a single unit test:

```bash
./gradlew testDebugUnitTest --tests "com.vadhod.apkextractor.core.util.FilenamesTest"
```

Development is on-device (the working device is a moto g45 5G / Android 15). There is no emulator flow
documented; verify real behavior by installing the debug APK.

## Non-negotiable constraints (read `rules.md` before adding anything)

These are enforced, not aspirational — violating them is a bug:

- **No `INTERNET` permission, ever.** Offline-only: no network code, telemetry, analytics, crash
  reporting, ads, or SDKs that phone home. `app/src/test/.../NoInternetPermissionTest.kt` is a guardrail
  test that fails the build if `INTERNET`/`ACCESS_NETWORK_STATE` appears in the manifest — do not disable it.
- **Pure non-root.** Only public `PackageManager` + Storage Access Framework (SAF). No `su`, Shizuku, or
  reflection into hidden APIs.
- **Least privilege.** Storage only via SAF — no `READ/WRITE_EXTERNAL_STORAGE`, no
  `MANAGE_EXTERNAL_STORAGE`. The only sensitive permission is `QUERY_ALL_PACKAGES` (justified: an app
  extractor's core job is enumerating installed apps). Adding any permission requires a note in
  `progress.md` and `architecture.md` §9.

## Dependency locking (the user's explicit requirement)

- **All versions live in `gradle/libs.versions.toml`** — no inline versions in `build.gradle.kts`.
  Critical libs use `version = { strictly = "x.y.z" }` so transitive deps can't downgrade them.
- **Never downgrade a pinned version.** `libraries-used.md` is the source of truth and records every
  dependency with a "verified on" date and justification. Upgrade only to a newer **stable** release,
  and re-verify the version online first (knowledge-cutoff numbers are stale) — then update the date and
  log it in `progress.md`.
- **No pre-release versions** (`-alpha`/`-beta`/`-rc`/snapshots) without an explicit recorded decision.
- Platform APIs are deliberately preferred over libraries (e.g. `java.util.zip` instead of zip4j,
  `CertificateFactory` instead of apksig, state-based nav instead of a nav library) to keep the
  dependency/attack surface minimal.

## Architecture

Layered by `core` / `data` / `feature` under `app/src/main/java/com/vadhod/apkextractor/`, structured so
it can later split into modules without churn. See `architecture.md` for the full blueprint.

- **`core/`** — framework-agnostic domain + design system.
  - `model/` — immutable domain types: `AppEntry` (+ `isSplit`), `SortOrder`, `ExportFormat`
    (`SingleBaseApk` / `BundleApks`), `ThemeMode`. No Android framework types leak into the domain.
  - `util/` — `DispatcherProvider`, `AppResult`/`AppError` (typed result wrapper via `appResult{}`),
    `Filenames` (sanitation), `Formatters`.
  - `design/` — **the design system**. `color/PastelColors`, `Gradients` (+ `LocalAppGradients`),
    `theme/` (`VadhodTheme`, `AppShapes`, Nunito `Type`), `components/` (reusable `GradientBackground`,
    `AppIcon`, `PillTabs`, `SearchField`, `AppListItem`, etc.).
- **`data/`** — I/O and repositories (single source of truth per concern, all off the main thread):
  - `packages/` — `PackageRepository` + `PackageManagerSource` (system/user classification, split paths,
    sizes) and the Coil `AppIconFetcher`/`Keyer` for offline icons.
  - `extract/` — the extraction pipeline (the part that must be bulletproof): `ApkExtractor` (single
    stream-copy), `SplitBundler` (`ZipOutputStream` → `.apks`), `ApkSink`/`SafApkSink` (SAF writer).
  - `inspect/` — `ApkInspector` (zip entries + signing SHA-256 via `CertificateFactory`).
  - `settings/` — `SettingsRepository` (DataStore Preferences; theme, bundle-default, persisted SAF tree URI).
  - `share/` — `Exporters` (FileProvider share intent, icon PNG export).
- **`feature/`** — MVVM screens; UI is a pure function of immutable `StateFlow` state, events flow up:
  - `applist/` — `AppListViewModel` (search, sort+persistence, multi-select, batch extract with per-item
    + overall progress), `AppListScreen`, `ExtractionSheet` (live % progress).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pgxor/com.vadhod.apkextractor](https://github.com/pgxor/com.vadhod.apkextractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
