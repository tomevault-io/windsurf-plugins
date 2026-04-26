---
trigger: always_on
description: Single source of truth for AI agents working on this project. Read this file completely before making any changes.
---

# CLAUDE.md — LuckFox Agent V2

Single source of truth for AI agents working on this project. Read this file completely before making any changes.

---

## 1. Project Overview

LuckFox Agent V2 is a voice-activated AI agent running on the **LuckFox Pico Max** (Rockchip RV1106 / ARM Cortex-A7, 256MB DDR3). It uses a dual-process architecture:

- **`luckfox_gui`** (C binary): drives LVGL v9 GUI on a 240×240 ST7789 SPI display, owns GPIO buttons, emits button events over IPC
- **`http_api_server_v2.py`** (Python): HTTP API on port 8080, AI pipeline coordination (STT → LLM → TTS via MacBook), drives C binary state via IPC

```
User presses CTRL button
        ↓
luckfox_gui (C) → IPC event → Python HTTP server
                                    ↓
                          HTTP → MacBook pipeline
                          (audio → STT → LLM → TTS)
                                    ↓
                    IPC command → luckfox_gui (C)
                                    ↓
                        LVGL → SPI → ST7789 240×240
```

### Why Dual-Process?

1. **Performance**: C + LVGL renders at native speed vs Python's ~40ms/frame
2. **LVGL threading**: LVGL is not thread-safe; isolated process avoids sync issues
3. **Stability**: Python crash doesn't kill GUI; display keeps showing last screen
4. **Separation**: GUI rendering vs API/network/audio are fundamentally different workloads

---

## 2. Current Status (2026-03-24)

**Display + all 9 buttons confirmed working on hardware.**

**IPC/HTTP integration complete.** All three Python files speak the V2 protocol:
- `gui_client.py`: `set_state()` sends `{"cmd": "set_state", "state": "...", "text": "..."}` matching `cmd_parser.c`
- `http_api_server_v2.py`: V2 agent state endpoints (`GET/POST /api/agent/state`), CTRL button events drive state machine (pressed→LISTENING, released→THINKING)
- `main.py`: launches `http_api_server_v2.py`

**IDLE screen confirmed working on hardware:**
- Page 0 (Status): time updates every second, date and IP display correctly
- Page 1 (Kawaii face): smooth animation, blink/bounce/emotion state machine working
- LEFT/RIGHT buttons navigate between pages
- Clock uses `time()`/`localtime()`/`strftime()` — pure C stdlib, no shell required

**Camera reworked (2026-03-20):**
- rkipc cycle snapshots **DISABLED** (`enable_cycle_snapshot = 0` in `/oem/usr/share/rkipc-300w.ini`)
- RTSP stream at native **25 FPS**, H.265/HEVC, 2304×1296, with PCM A-law 8kHz audio
- RTSP exposed externally via FRP: `rtsp://luckfoxpico1.aiserver.onmobilespace.com:8554/live/0`
- `/api/capture` uses ffmpeg `select=key` filter → proper IDR keyframe JPEG (~10s response)
- `/api/stream` removed — use RTSP directly in VLC/ffplay
- Static ffmpeg 7.0.2 armhf at `/mnt/sdcard/ffmpeg` (32MB, gitignored, in sync.sh)
- SD card remounted with `fmask=0022` (exec enabled) via udev rule change

**Audio input — INMP441 mic integrated (2026-03-24):**
- INMP441 I2S MEMS microphone wired to ESP32-C3 GPIO10 (I2S DIN)
- Full-duplex I2S: speaker (MAX98357A) and mic (INMP441) share BCLK (GPIO0) / LRC (GPIO1)
- ESP32-C3 firmware: `luckfox_audio_fulldup.ino` — captures 32-bit I2S, converts to 16-bit PCM (`raw32[i] >> 8`), streams upstream via `PKT_MIC_DATA` over UART2
- Python `MicReceiver` class in `audio_sender.py` — background thread reads mic packets, buffers PCM, exports WAV on demand
- Mic lifecycle tied to `AUDIO_START`/`AUDIO_STOP` — mic starts/stops with speaker session
- HTTP endpoints: `/api/audio/record/start`, `/api/audio/record/stop`, `/api/audio/record/download`, `/api/audio/stream`
- Bandwidth: 512 kbps bidirectional audio (mic + speaker) within UART's 736 kbps capacity

**Live stream client fixed (2026-03-24):**
- `luckfox_remote.py stream` uses two separate ffplay processes (video RTSP + audio HTTP)
- Previous pipe-mux approach (ffmpeg → NUT → ffplay) failed due to rkipc's ~15s GOP — P-frames flooded pipe before IDR keyframe
- Direct RTSP → ffplay handles long GOP correctly (same as VLC)

**Pending — next actions:**

1. **MacBook AI pipeline** — receive recorded audio from board, run STT (speech-to-text), send transcript to LLM, get response, run TTS (text-to-speech), send WAV back to board
2. **Board playback** — receive TTS WAV via `/api/audio/play`, play through ESP32-C3, set SPEAKING state with response text, return to IDLE when playback completes
3. **Error handling** — on any pipeline failure (network timeout, STT error, LLM error), set ERROR state with message, then return to IDLE after a few seconds
4. **End-to-end test** — press CTRL, speak, release CTRL → board records → MacBook processes → board speaks response → IDLE

---

## 3. Agent State Machine

Five states. The C binary renders the correct screen for each state. Python drives state transitions via IPC.

| State | Trigger | Visual |
|-------|---------|--------|
| `IDLE` | Boot / pipeline complete | Logo + "Press CTRL to start" |
| `LISTENING` | CTRL pressed | Mic/waveform animation, green |
| `THINKING` | CTRL released | Spinner/dots, orange |
| `SPEAKING` | LLM response ready | Response text + waveform, cyan |
| `ERROR` | Any failure | Error message, red |

```
IDLE ──[CTRL press]──► LISTENING ──[CTRL release]──► THINKING

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/josedamianm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
