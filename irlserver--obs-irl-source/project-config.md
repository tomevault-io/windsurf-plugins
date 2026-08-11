---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

IRL Source is a third-party OBS Studio plugin (C11, AGPL-3.0) for receiving live IRL streams over SRT, RTMP, or any FFmpeg-supported protocol. It solves IRL-specific problems: audio jitter buffering, PTS discontinuity repair, adaptive playback speed, keyframe gating, hardware-accelerated decoding, and mid-stream resolution changes.

## Build commands

The plugin statically links its own FFmpeg, libsrt and mbedTLS (see `deps/README.md`), so the first step on every platform is building that stack. It is incremental, so this is a one time cost per version bump.

### Linux

```bash
sudo apt install build-essential cmake pkg-config nasm libobs-dev libva-dev
./deps/build-deps.sh
cmake -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build --parallel
./scripts/verify-plugin.sh build/obs-irl-source.so
```

### Windows (MSVC)

`deps/build-deps.sh` runs inside MSYS2 with the MSVC environment active (FFmpeg's configure needs a POSIX shell even when driving `cl.exe`). See the `windows-x64` job in `.github/workflows/build.yml` for the exact setup.

```powershell
cmake -B build -G "Visual Studio 18 2026" -A x64 -DOBS_SOURCE_DIR=obs-src
cmake --build build --config RelWithDebInfo
```

### macOS (Apple Silicon)

```bash
brew install cmake pkg-config nasm simde uthash jansson
./deps/build-deps.sh
cmake -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo -DOBS_SOURCE_DIR=$PWD/obs-src \
    -DCMAKE_DISABLE_FIND_PACKAGE_PkgConfig=ON
cmake --build build --parallel
./scripts/verify-plugin.sh build/obs-irl-source.so
```

Output: `build/obs-irl-source.so` (Linux/macOS) or `build/RelWithDebInfo/obs-irl-source.dll` (Windows).

`-DIRL_BUNDLED_FFMPEG=OFF` falls back to linking a system or obs-deps FFmpeg. That path still works for a quick compile check, but it reintroduces the per OBS line binding the bundled stack exists to remove, so it is not what releases use.

`scripts/verify-plugin.sh` is not optional polish. It asserts the two properties that make the bundled stack correct and that a successful compile does not prove: that the binary carries no `libav*` dependency, and that it exports nothing but `obs_module_*`. CI runs it (and a `dumpbin` equivalent on Windows) on every build.

There are no tests.

## Architecture

Single OBS MODULE shared library. All source is C11.

### Data flow

```
[receiver thread]: FFmpeg URL, demux, decode, PTS repair
  audio: resample, write to jitter buffer
  video: keyframe gate, push decoded frame (PTS in ns) onto video queue

[video thread]: pop video queue, HW frame transfer, format conversion,
                OBS async video output

[audio thread]: drain jitter buffer, speed correction, concealment,
                OBS audio output
```

### Audio output contract (verified against libobs source)

The audio core is built around three facts about libobs:

1. OBS timestamps must be contiguous (`ts[n+1] = ts[n] + frames/rate`). Deviations under 70ms are smoothed, 70ms to 2s gaps are zero filled by OBS (audible), larger jumps flush all queued audio. The plugin therefore derives timestamps from a pure sample counter anchored once at prime time and never jumps the clock outside declared restarts.
2. Changing `samples_per_sec` between submissions makes OBS destroy and recreate its per source resampler with no crossfade (a click per change). Playback speed is instead applied inside the plugin with a persistent swresample compensation, and the rate submitted to OBS never changes.
3. The OBS mixer consumes 21.3ms ticks against wall clock. A source whose queued audio runs dry gets a tick of silence plus a time shifted splice (crackle), and a source that falls behind the mix window causes OBS to permanently add global audio buffering. After priming, the pump always emits (real audio or shaped concealment silence) and keeps a fixed lead ahead of wall clock.

Buffer regulation happens through playback speed only, asymmetric like IRLToolkit's player: builds at an inaudible -2%, drains post-stall backlog at up to +5% (mild chipmunk). Content is never skipped once playback has primed. Backlog beyond a fill ceiling is pushed back into the transport by pausing the read loop (TCP/RTMP backpressure; SRT bounds itself via its latency window), and startup backlog is trimmed only before priming.

### Source files

- **`src/plugin.c`**: OBS module entry point. Registers `irl_source_info` with callbacks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [irlserver/obs-irl-source](https://github.com/irlserver/obs-irl-source) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
