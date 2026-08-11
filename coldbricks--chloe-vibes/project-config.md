---
trigger: always_on
description: These rules are non-negotiable. Violations require stopping and asking the user.
---

# CRITICAL PROJECT DIRECTIVES

These rules are non-negotiable. Violations require stopping and asking the user.

- NEVER commit, push, deploy, or run destructive commands without explicit user approval
- NEVER create files unless the task strictly requires it — prefer editing existing files
- NEVER guess at architecture — read the code first, form a model, then act
- When an error occurs, read the FULL message. Trace the cause. Do not blame the platform or say "can't be done" without exhaustive investigation

# Project Identity

- **Name:** ChloeVibes
- **Type:** Audio-reactive haptic controller — Android app (Kotlin/Compose) + Windows desktop (Rust/egui)
- **Language/Stack:** Kotlin + Jetpack Compose (Android), Rust + eframe/egui (desktop)
- **Build System:** Gradle 9.0.0 / Kotlin 2.1.0 (Android), Cargo (Rust desktop)
- **Target Platform:** Android 8.0+ (API 26) targeting API 35; Windows x86_64 (Rust desktop)
- **Repo Root:** (use current working directory)
- **Primary Branch:** master

# Key File Map

Consult these before searching blindly. Paths are relative to repo root.

| Purpose | Path |
|---|---|
| Android entry point | `android/app/src/main/kotlin/com/ashairfoil/chloevibes/MainActivity.kt` |
| Audio capture | `android/app/src/main/kotlin/com/ashairfoil/chloevibes/audio/AudioCaptureManager.kt` |
| Spectral analysis | `android/app/src/main/kotlin/com/ashairfoil/chloevibes/audio/SpectralAnalyzer.kt` |
| Noise gate | `android/app/src/main/kotlin/com/ashairfoil/chloevibes/audio/Gate.kt` |
| Beat detection | `android/app/src/main/kotlin/com/ashairfoil/chloevibes/audio/BeatDetector.kt` |
| ADSR envelope | `android/app/src/main/kotlin/com/ashairfoil/chloevibes/audio/EnvelopeProcessor.kt` |
| Climax modulation | `android/app/src/main/kotlin/com/ashairfoil/chloevibes/audio/ClimaxEngine.kt` |
| Presets | `android/app/src/main/kotlin/com/ashairfoil/chloevibes/audio/Presets.kt` |
| BLE device manager | `android/app/src/main/kotlin/com/ashairfoil/chloevibes/device/BleDeviceManager.kt` |
| Lovense protocol | `android/app/src/main/kotlin/com/ashairfoil/chloevibes/device/LovenseProtocol.kt` |
| Main UI screen | `android/app/src/main/kotlin/com/ashairfoil/chloevibes/ui/MainScreen.kt` |
| Theme | `android/app/src/main/kotlin/com/ashairfoil/chloevibes/ui/Theme.kt` |
| Android manifest | `android/app/src/main/AndroidManifest.xml` |
| App build config | `android/app/build.gradle.kts` |
| Root build config | `android/build.gradle.kts` |
| Rust entry point | `src/main.rs` |
| Rust signal engine | `src/audio.rs` |
| Rust GUI + pipeline | `src/gui.rs` |
| Rust presets | `src/presets.rs` |
| Rust settings | `src/settings.rs` |
| Rust utilities | `src/util.rs` |
| Rust build config | `Cargo.toml` |
| Tests | `cargo test` (27 unit tests for audio, util), plus real hardware testing |
| CI/CD | GitHub Actions (.github/workflows/ci.yml) — Rust build/test/lint + Android build |
| Generated (DO NOT EDIT) | `android/build/`, `android/app/build/`, `android/.gradle/`, `target/` |

# Build and Run

```
# Android — build debug APK
cd android && ./gradlew assembleDebug

# Android — install to connected device
adb install -r android/app/build/outputs/apk/debug/app-debug.apk

# Android — clean
cd android && ./gradlew clean

# Rust desktop — build
cargo build --release

# Rust desktop — run
cargo run --release
```

Always run the build after making changes. If the build fails, fix it before moving on. Never hand the user a broken build.

# Architecture Invariants

These are load-bearing design decisions. Do not refactor away from them without explicit approval.

1. Signal chain order is fixed: SpectralAnalyzer -> Gate -> BeatDetector -> EnvelopeProcessor -> ClimaxEngine -> Output. Never reorder or skip stages.
2. Audio processing runs on a dedicated thread at ~60Hz. UI reads state via volatile fields. Never do signal processing on the main/UI thread.
3. The Kotlin signal engine is a direct port of the Rust engine (`src/audio.rs`). When modifying one, consider whether the other needs the same change to stay in sync.
4. BLE commands go through LovenseProtocol for command formatting and BleDeviceManager for transmission. Never write raw BLE commands outside this path.
5. All Lovense commands are ASCII strings terminated with semicolons, sent via Nordic UART Service (NUS). The intensity range is 0-20, not 0-100 or 0-255.
6. Presets are immutable snapshots of all signal processing parameters. When adding a parameter, it must be included in the Preset data class and all existing presets updated.

# Coding Standards

## Style Rules
- 4-space indentation, no tabs (Kotlin). Standard rustfmt conventions (Rust).
- Preserve existing formatting in files you edit — match the style already present.
- No wildcard imports in Kotlin.
- No trailing summaries or recap comments.

## Naming Conventions
- Files: PascalCase.kt (Kotlin), snake_case.rs (Rust)
- Variables: camelCase (Kotlin), snake_case (Rust), constants: SCREAMING_SNAKE (both)
- Boolean vars prefixed: is, has, should, can
- Signal processing parameters match across Kotlin and Rust (e.g., `attackMs`, `gateThreshold`)

## Patterns to Use
- Volatile fields for cross-thread state sharing (Android). Arc<Mutex<>> or SharedF32 for Rust.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coldbricks/Chloe-Vibes](https://github.com/coldbricks/Chloe-Vibes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
