---
trigger: always_on
description: This file provides guidance to AI coding agents working with this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with this repository.

## Build Commands

```bash
# Build
./gradlew build

# Unit tests (debug config flavor)
./gradlew testDebugConfigDebugUnitTest

# Instrumented tests (requires connected device/emulator)
./gradlew connectedDebugConfigDebugAndroidTest

# Coverage reports
./gradlew jacocoTestReport          # Unit test coverage → app/build/reports/jacoco/html/
./gradlew jacocoFullReport          # Combined coverage → app/build/reports/jacoco/htmlFull/
./gradlew jacocoTestCoverageVerification  # Enforce 5% line coverage minimum
```

**Build flavors:** `debugConfig` (debug, signed) and `releaseConfig` (minified/shrunk). Most tasks require the flavor in the task name (e.g., `testDebugConfigDebugUnitTest`).

## Architecture Overview

### Dual Codebase (V1 → V2 Migration in Progress)

The app contains two parallel implementations:
- **V2** (`app/src/main/java/.../v2/`) — active development target: Kotlin, Jetpack Compose, Hilt, Room, MVVM + Clean Architecture
- **V1** (`app/src/main/java/.../` root level) — legacy: Java, Views, manual DI via `Injector.java`, MVP pattern

The user can toggle between them via settings. `HomeActivity` is the V2 entry point; `MainActivity` is the V1 entry point.

**All new development should target the `v2/` package.**

### V2 Package Structure

```
v2/
├── app/          # UI layer — one subfolder per screen (home, records, settings, deleted, lostrecords, info, welcome)
│   └── components/   # Reusable Compose components
├── audio/        # Recording and playback logic + foreground services
├── data/         # Data layer — Room DB, data sources, file I/O, prefs
│   ├── room/     # AppDatabase, DAOs, entities
│   ├── model/    # Domain models (Record, SortOrder, etc.)
│   └── Mappers.kt
├── navigation/   # RecorderNavigationGraph.kt + Routes.kt
├── di/           # Hilt modules (AppModule, DatabaseModule, DataSourceModule) + qualifiers
└── theme/        # Compose theme (dark/dynamic color)
```

### Core Patterns

- **Language/UI:** Kotlin, Jetpack Compose, Material3
- **DI:** Hilt (`@HiltAndroidApp`, `@AndroidEntryPoint`, `@HiltViewModel`)
- **Database:** Room
- **Async:** Coroutines with `@IoDispatcher` / `@MainDispatcher` qualifiers
- **Navigation:** Navigation Compose
- **State management:** ViewModels expose immutable state data classes via `mutableStateOf()`. One-shot events use `SharedFlow`.

### Service-Based Audio Architecture

Recording and playback run as **foreground services** (`AudioRecordingService`, `AudioPlaybackService`). ViewModels bind via `ServiceConnection` and receive state via `StateFlow`/`SharedFlow`.

### Entry Points

- `HomeActivity` — V2 entry point
- `MainActivity` — V1 legacy entry point

### Key Non-Obvious Behaviors

- **Broken record recovery:** `BrokenRecordRestorer.kt` detects/repairs WAV/M4A/3GP files interrupted by crash or reboot.
- **Waveform data:** See the dedicated [Waveform Visualization](#waveform-visualization) section below.
- **Bluetooth mic:** `AudioManagerHelper` monitors Bluetooth state reactively; `HomeViewModel` manages source selection.
- **Preferences:** `PrefsV2` wraps SharedPreferences with Flow-based reactive updates. `Prefs` and `PrefsV2` share the same SharedPreferences file.

## Waveform Visualization

The waveform UI has two distinct lifecycles: a **live** waveform that scrolls during active recording, and a **static** waveform rendered from a persisted amplitude array for finished records. Both V1 and V2 follow the same conceptual pipeline but with different recorder/UI stacks. The post-recording decoder is **shared** between V1 and V2.

### Live waveform (during recording)

The live waveform deliberately keeps in memory **only the slice currently visible on screen**. Older amplitudes that have scrolled off the left edge are dropped — they are not part of any persistent buffer. The full waveform of the finished file is reconstructed afterwards by decoding the audio (see next section).

**V2 — `v2/audio/`**

- `MediaRecorderBase.kt` polls `mediaRecorder.getMaxAmplitude()` from a scheduled `recordingTimeUpdateRunnable`, accumulating samples into `IntArrayList amplitudesBuffer`. The timer emits `RecorderEvent.OnRecordingProgress(durationMills, amplitude)` every `AppConstants.RECORDING_VISUALIZATION_INTERVAL_NEW` (~10ms).
- `AudioRecordingService.kt` keeps a fixed-size sliding window in `LinkedList<Int> recordingAmplitudes`. The size cap is computed from screen width by `calculateRecordingAmplitudeBufferSize()` (`AudioRecordingService.kt:781`) so the buffer holds exactly the half-screen-worth of samples that are visible on the right side of the scrubber. On every new sample, `handleRecordingProgress()` appends to the tail; once the cap is exceeded it calls `recordingAmplitudes.removeFirst()` — this is the "delete what's no longer visible" behavior. Samples are also scaled by ~1.2× for visual amplification.
- The service publishes `RecordingServiceState.amplitudes: IntArray` plus `totalSampleCount` / `waveformDataOffset` (absolute timeline position) via `StateFlow`. `HomeViewModel` collects this and feeds the Compose state.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dimowner/AudioRecorder](https://github.com/Dimowner/AudioRecorder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
