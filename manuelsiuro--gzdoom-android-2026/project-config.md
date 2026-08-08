---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An Android port of the GZDoom engine bundled with the open-source Freedoom assets, published as "Freedoom for Android" (`com.msa.freedoom`). It is a fork of nvllsvm's GZDoom-Android. The app is a thin Java/Kotlin shell around a large C/C++ engine compiled via the NDK.

## Build

The native engine and the Android app are built in **two separate, sequential steps**. Gradle does *not* drive the native build (there is no `externalNativeBuild` block); instead `ndk-build` produces `.so` files into `doom/src/main/libs`, which Gradle picks up via `jniLibs.srcDirs("src/main/libs")`.

```bash
ANDROID_NDK_HOME=~/Library/Android/sdk/ndk/27.0.12077973 ./build_native.sh   # ndk-build → .so
./gradlew :doom:assembleDebug                                                # APK = Kotlin + .so
```

- `build_native.sh` auto-locates `ndk-build` (honours `ANDROID_NDK_HOME`, else the highest NDK under the SDK) and runs it with `jni/Application.mk`. The committed `.so` under `doom/src/main/libs/{armeabi-v7a,arm64-v8a}/` mean the app **already assembles + runs without re-running the native build**; only re-run it if you change `doom/src/main/jni/**`.
- Native target ABIs are **`armeabi-v7a` and `arm64-v8a`** (`doom/src/main/jni/Application.mk`).
- Toolchain: **Java 17** (`gradle.properties` pins `org.gradle.java.home` to a local Temurin 17 path — adjust per machine); **NDK r27** for the native build. `local.properties` (gitignored) must point `sdk.dir` at your SDK.
- `Application.mk` carries three load-bearing settings: `APP_STL := c++_shared`, `-DNO_SEC` (disables the Delta Touch licence-check code in the vendored glue), and **`-Wl,-z,nostart-stop-gc`** — GZDoom registers CVARs/classes/actions in custom ELF sections (`areg`/`creg`/`vreg`/…) walked via `__start_*`/`__stop_*`; modern lld + the NDK's default `--gc-sections` strips those entries silently and the engine crashes on the first CVAR access. Do not remove that flag.
- Unit tests: `./gradlew :doom:testDebugUnitTest` (guards the launch command line). JNI seam signatures are golden-tested: `bash .jni-golden/verify.sh` after a debug build (`--update` to regenerate).

## The native engine (UZDoom 5.0.0-pre, emileb's uz_5.0_pre / GZDoom 4.15 fork)

The native layer is **rebased onto emileb's maintained UZDoom mobile port** (UZDoom 5.0.0-pre, [`github.com/emileb/gzdoom`](https://github.com/emileb/gzdoom) branch `uz_5.0_pre`, a GZDoom 4.15-derived Delta Touch engine) plus his OpenTouch support libraries. The engine reports `GAMENAME "UZDoom"` / `VERSIONSTR "5.0.0-pre"` (see `engine/gzdoom/src/version.h`). Everything is **vendored** under `doom/src/main/jni/` (no submodules):

- `engine/gzdoom/` — the engine (nested one level so the glue's `../../../Clibs_OpenTouch/...` relative paths resolve to the jni root). Its `mobile/Android.mk` pulls in `lzma`, `bzip2`, `glslang`, `zwidget`, ZMusic (from `libraries/ZMusic`) and the engine module `gzdoom`.
- `Clibs_OpenTouch/` — emileb's shared JNI glue (`android_jni_inc.cpp`, `idtech1/` game interface, `port_act_defs.h`). The Google Play licence check lives behind `#ifndef NO_SEC` (we define `NO_SEC`). Also provides `logwritter` and the prebuilt `vpx_player`.
- `SDL2_OpenTouch/` — SDL 2.0.12 fork built with `OPENTOUCH_SDL_EXTRA` (beloko extras: swap-buffer callback, mouse injection; Java prefix `org.libsdl.app2012`). Plus `SDL2_net`.
- `MobileTouchControls/` — current touch-controls library (+ libpng/libzip/sigc++/TinyXML). Calls back into `com.beloko.touchcontrols.*` and `org.libsdl.app.NativeConsoleBox` Java by name.
- `AudioLibs_OpenTouch/` — `openal` (source build, OpenSL backend), `libsndfile`, `libmpg123`, `fluidsynth-lite`, flac (ZMusic deps). No FMOD.
- `SAFFAL/` — scoped-storage file-access layer (libsaffal + Java `com.opentouchgaming.saffal`, vendored into the app).
- `jpeg8d/`, `Android_webp.mk` (builds `webpmux` from `engine/gzdoom/libraries/webp` — our makefile, upstream only ships CMake).

Local patches to the vendored engine (grep for these before re-vendoring a newer branch): `ALooper_pollAll→ALooper_pollOnce` (NDK r27, SDL sensor), `-DUZDOOM` (selects the `Mobile_IN_Move(usercmd_t*)` glue signature), crashcatcher disabled on Android (`i_main.cpp` — it fork()s a debugger and deadlocks), ES 3.1→3.0 context fallback in `sdlglvideo.cpp`, module renamed `g`→`gzdoom`, `core_shared` dropped from the link line (not needed), webp include path fixed.

**Engine data**: UZDoom 5.0 (`__MOBILE__`) loads `<game_path>/res/uzdoom.pk3` (BASEWAD) + `res/uzdoom_game_support.pk3` (OPTIONALWAD, carries IWADINFO). These are built from the engine's own `wadsrc*/static` trees (plain zips) and bundled in `doom/src/main/assets/` together with `lights.pk3`, `brightmaps.pk3`, `game_widescreen_gfx.pk3` and the engine's `gzdoom.sf2`; `LaunchState.launchGame()` unpacks them. If you bump the engine, rebuild the pk3s from the matching `wadsrc` or the engine errors at boot. Touch-control art is `doom/src/main/assets/*.png` + `font_dual.png` (a generated 16×16-cell glyph atlas; glyph spacing is auto-derived from alpha).

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manuelsiuro/GZDoom-Android-2026](https://github.com/manuelsiuro/GZDoom-Android-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
