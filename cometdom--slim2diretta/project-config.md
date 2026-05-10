---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md - slim2diretta

This file provides guidance to Claude Code when working with this repository.

## Project Overview

**slim2diretta** is a native LMS (Slimproto) player with Diretta output in a mono-process architecture. It replaces the couple squeezelite + squeeze2diretta-wrapper with a single binary that implements Slimproto directly and feeds DirettaSync without an intermediate pipe.

**License**: MIT (no GPL code copied). Slimproto protocol implemented from public documentation.

## Build Commands

```bash
# Build (auto-detects architecture and SDK)
mkdir build && cd build
cmake ..
make

# Specific architecture variants
cmake -DARCH_NAME=x64-linux-15v3 ..       # x64 AVX2 (most common)
cmake -DARCH_NAME=aarch64-linux-15 ..     # Raspberry Pi 4
cmake -DARCH_NAME=aarch64-linux-15k16 ..  # Raspberry Pi 5 (16KB pages)

# Custom SDK path
export DIRETTA_SDK_PATH=/path/to/DirettaHostSDK_148
cmake ..

# Clang + LTO (recommended for best audio quality)
CC=clang CXX=clang++ cmake -DENABLE_LTO=ON ..
```

## Running

```bash
# List available Diretta targets
sudo ./slim2diretta --list-targets

# Basic usage
sudo ./slim2diretta -s <lms-ip> --target 1

# With player name and verbose
sudo ./slim2diretta -s <lms-ip> --target 1 -n "Living Room" -v

# With CPU affinity (cores 2 and 3 isolated via isolcpus=2,3)
sudo ./slim2diretta --target 1 --cpu-audio 2 --cpu-other 3
```

## Architecture

```
LMS (network)
  -> slim2diretta (single process)
    -> SlimprotoClient (TCP port 3483) : control
    -> HttpStreamClient (port 9000) : encoded audio stream
    -> Decoder (FLAC/PCM/DSD — native or FFmpeg backend)
    -> DirettaSync (ring buffer + SDK)
      -> Diretta Target (UDP/Ethernet)
        -> DAC
```

**Threading**: main (init/signals) + slimproto (TCP LMS) + audio (HTTP->decode->push) + SDK worker (DirettaSync internal)

**CPU affinity** (`--cpu-audio`, `--cpu-other`): optional thread pinning via `pthread_setaffinity_np`, default `-1` (no pinning). `--cpu-audio` pins the SDK worker thread and is also passed to `DIRETTA::Sync::open(cpuMain, cpuOther, ...)`; the `OCCUPIED` flag (bit 16) is added to threadMode automatically when `cpuAudio >= 0`. `--cpu-other` pins the main thread, the audio (HTTP→decode→push) thread, and the Slimproto receive thread. Both are exposed via CLI and Web UI (CPU Affinity group). Aligned with DirettaRendererUPnP.

**Startup**: Both Diretta target discovery and LMS autodiscovery retry indefinitely:
- `discoverTarget()` retries every 2s (log every 5s) until found or cancelled. Pass `std::atomic<bool>* stopSignal` to `enable()` to activate retry mode.
- LMS autodiscovery (when no `-s` is specified) retries every 2s in a `while(g_running)` loop until LMS responds. Ctrl+C cancels cleanly.

**Key Components**:

| File | Purpose |
|------|---------|
| `src/main.cpp` | Entry point, CLI, signal handling, logging init |
| `src/Config.h` | Configuration struct |
| `src/PlayerController.cpp/h` | Orchestrator: state machine, thread coordination |
| `src/SlimprotoClient.cpp/h` | Slimproto TCP protocol client |
| `src/SlimprotoMessages.h` | Binary protocol message structs |
| `src/HttpStreamClient.cpp/h` | HTTP audio stream fetcher |
| `src/Decoder.h` | Decoder abstract interface |
| `src/FlacDecoder.cpp/h` | FLAC decoder (libFLAC) |
| `src/PcmDecoder.cpp/h` | PCM/WAV/AIFF header parser |
| `src/Mp3Decoder.cpp/h` | MP3 decoder (libmpg123, optional) |
| `src/OggDecoder.cpp/h` | Ogg Vorbis decoder (libvorbisfile, optional) |
| `src/AacDecoder.cpp/h` | AAC decoder (fdk-aac, optional) |
| `src/FfmpegDecoder.cpp/h` | FFmpeg decoder backend (libavcodec, optional) |
| `src/DsdProcessor.cpp/h` | DSD conversions (interleaved->planar, DoP->native) |
| `diretta/DirettaSync.cpp/h` | Diretta SDK wrapper (shared with squeeze2diretta) |
| `diretta/DirettaRingBuffer.h` | Lock-free SPSC ring buffer |
| `diretta/globals.cpp/h` | Logging configuration |
| `diretta/LogLevel.h` | Centralized log level system |
| `diretta/FastMemcpy*.h` | SIMD memory operations |

**Web UI** (`webui/`):

| File | Purpose |
|------|---------|
| `webui/diretta_webui.py` | HTTP server (custom BaseHTTPRequestHandler, no framework) |
| `webui/config_parser.py` | Config parsers: `ShellVarConfig` (KEY=VALUE) and `CliOptsConfig` (CLI args) |
| `webui/profiles/slim2diretta.json` | Product profile defining settings groups and field types |
| `webui/templates/index.html` | HTML template with embedded JavaScript |
| `webui/static/style.css` | Minimal CSS styling |
| `webui/slim2diretta-webui.service` | Systemd service (port 8081) |

**Startup & Install**:

| File | Purpose |
|------|---------|
| `start-slim2diretta.sh` | Startup wrapper: reads `/etc/default/slim2diretta`, applies priority, `eval exec` |
| `install.sh` | Interactive installer (binary, service, webui) |
| `slim2diretta.service` | Main systemd service |

## Code Style

- **C++17** standard
- **Classes**: `PascalCase`
- **Functions**: `camelCase`
- **Members**: `m_camelCase`
- **Constants**: `UPPER_SNAKE_CASE`
- **Globals**: `g_camelCase`
- **Indentation**: 4 spaces

## Slimproto Protocol


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cometdom/slim2Diretta](https://github.com/cometdom/slim2Diretta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
