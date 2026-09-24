---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A native-C++ Android port of the [spektrafilm](https://github.com/andreavolpato/spektrafilm)
spectral film-simulation engine, driven by a Jetpack Compose UI. The engine reconstructs spectra
from RGB and runs a physically-based virtual **negative → enlarger → print → scan** pipeline.
GPLv3 (derivative of GPLv3 spektrafilm).

**The prime directive is bit-exact parity with the upstream spektrafilm oracle.** Every engine
stage was ported parity-first against golden vectors captured from the real Python engine. Any
change to `engine/spektra-core/src/main/cpp/**` must keep the host parity suite green (see below).
"Bit-exact" = within parity tolerance (`max_abs ≤ 1e-4`, `rms ≤ 1e-5`) of the oracle **and**
byte-identical across thread counts — not necessarily byte-identical across CPU architectures
(`-ffast-math` FMA contraction differs by arch).

## Module layout

Gradle modules actually built (`settings.gradle.kts`):
- **`:app`** — `com.spectrafilm.app`, the application. All UI lives here (~82 Kotlin files in
  `app/src/main/java/com/spectrafilm/app/`): `MainActivity`, the Lightroom-style editor
  (`Viewer`, `ParamsState`, `ImagePipeline`, `CropOverlay`, `CategoryIcons`/`SpectraIcons`),
  presets/recipes, settings, profile-curve browser, diagnostics.
- **`:engine:spektra-core`** — `com.spectrafilm.engine`. NDK C++ engine (`libspektra.so`) + the
  Kotlin facade `SpektraEngine` / `SpektraParams`. Bundles film/paper profiles, spectral LUTs,
  and ICC profiles under `src/main/assets/spektra/`.
- **`:lib:libraw`** — `libsfraw.so`, LibRaw via an ACES intermediate → linear ProPhoto RGB for RAW/DNG import.
- **`:lib:tiffwriter`** (`libsftiff.so`) and **`:lib:pngwriter`** (`libsfpng.so`) — 16-bit
  TIFF/PNG export writers.

`feature/film-emulation/` was a never-compiled pseudo-module (never in `settings.gradle.kts`);
it was deleted by #184 — its history survives in git and in `docs/DECISION.md`. The
real app is the standalone `:app` module documented in `docs/ARCHITECTURE.md`. The abandoned
ImageToolbox-host proposal survives only as historical decision input in `docs/DECISION.md`.
Start at `docs/EXECUTION_INDEX.md` for the current authority order and live-work protocol.

## Engine architecture (C++, `engine/spektra-core/src/main/cpp/`)

- **`spektra_jni.cpp`** — JNI bridge; the single native boundary. Buffers cross as direct
  `ByteBuffer` (interleaved float32 RGB, row-major) to avoid per-pixel JNI calls.
- **`spektra.cpp` / `spektra.h`** — top-level `simulate` / `simulate_preview` orchestration.
- **`runtime/stages/`** — the pipeline stages in order: `filming` (RGB → spectral via Hanatos2025
  LUT → camera raw → film density CMY, with DIR couplers), `printing` (film CMY → enlarger
  dichroic Y/M/C filters → print paper density), `scanning` (density → spectral radiance → CIE
  XYZ → output RGB), plus `crop_resize` and `autoexposure` geometry/metering stages.
- **`model/`** — photographic math: `spectral`, `density_curves`, `emulsion`, `couplers`,
  `diffusion` (halation + in-emulsion scatter), `grain` (Poisson-binomial particle model),
  `color_filters`, `color_output`, `glare`.
- **`kernels/`** — hot numeric primitives: `spectral_upsampling`, `gaussian`/`exponential_filter`
  (spatial convs), `interp`/`lut3d`, `stats` (samplers), `exp10.h` (vector `exp10` → NEON `fmla`
  on arm64, replaces `pow(10,−x)` in the spectral integrals), and `parallel` (deterministic
  fork-join per-pixel threading — output is byte-identical for any worker count).
- **`profiles/`** + **`io/npy_lut.cpp`** — profile JSON + `.npy`/`.lut` asset loaders.

Two quality modes mirror upstream: **preview** (downscaled, default 640px, for interactive
tuning) and **scan** (full-res, for export). Decode + simulate run off the main thread.

## Build commands

Required toolchain: **JDK 21** (Gradle `9.5.1` / AGP `9.3.2` with built-in Kotlin `2.2.10`; the PATH JDK 26 on
the laptop breaks Gradle, use Android Studio's JBR), **NDK r28c (`28.2.13676358`)**,
**CMake 3.22.1**, **build-tools 36.0.0** (AGP 9.3 minimum; also the `zipalign -P 16` / `apksigner` used by
the 16 KB and signing gates).
`sdkmanager "ndk;28.2.13676358" "cmake;3.22.1" "build-tools;36.0.0"`.

```bash
# Debug APK (builds libspektra/libsfraw/libsftiff/libsfpng .so for all 3 ABIs)
ANDROID_SDK_ROOT=/opt/android-sdk JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64 \
  ./gradlew :app:assembleDebug

# JVM unit tests (plus the androidTest instrumentation the android-emulator CI job runs)
./gradlew :app:testDebugUnitTest

# Lint (abortOnError = true; baseline at app/lint-baseline.xml)
./gradlew :app:lint
```

16 KB page check (CI gates this): `build-tools/36.0.0/zipalign -c -P 16 4 <apk>` must pass, and
every `arm64-v8a`/`x86_64` `.so` must have `0x4000` `LOAD` alignment (`readelf -lW`).

## Engine host-parity tests (the real gate)

Stage tests live in `engine/spektra-core/src/main/cpp/tests/` and run on the **host** g++
toolchain (not NDK) — they are not part of the Android library. After any engine change, run them.
Compile a single test against the full source set (note `-pthread` is required for
`kernels/parallel`):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thetechgeekko/Spektrafilm-android](https://github.com/thetechgeekko/Spektrafilm-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
