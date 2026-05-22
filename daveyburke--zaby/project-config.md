---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Zaby is an AI-powered animatronic teddy bear. The heavy lifting (Google Cloud Speech-to-Text, Gemini 3 Flash agent, Google Cloud Text-to-Speech) runs on a GCP Cloud Run server. The Raspberry Pi 5 in the bear's backpack streams microphone PCM to the server over a WebSocket and plays back synthesized PCM as it arrives, while driving motors for envelope-tracked mouth movement and neck rotation.

## Development Commands

### Pi-side setup
```bash
python -m venv zaby-env
source zaby-env/bin/activate
pip install -r requirements.txt
export ZABY_SERVER_URL=https://zaby-server-xxx.run.app
python main.py
```

### Server deploy
```bash
cd cloud_run
GEMINI_API_KEY=<key> ./deploy.sh
```

### Mac-side smoke test (no GPIO)
```bash
export ZABY_SERVER_URL=https://zaby-server-xxx.run.app
python test_client.py [speak|converse|both]
```

### Systemd service (Pi)
```bash
sudo systemctl start zaby.service
sudo systemctl stop zaby.service
sudo journalctl -u zaby.service -f
```
`/etc/zaby.env` must contain `ZABY_SERVER_URL=...`.

## Architecture

### Pi side (`main.py`, `conversation_client.py`, `bear_state.py`, `bear_animatronics.py`)

**main.py** — entry point. Generates a fresh `client_id` (UUID) each boot and sends it on the WS. The server no longer keys *agent state* by it (single-user / single-bear), but does use a change in `client_id` as a "Pi rebooted" signal to wipe in-RAM conversation history. Long-term memory in `MEMORY.md` survives the reset. Wires up the animatronics, `ConversationClient`, and `BearOnOffState`, then runs the main loop.

**conversation_client.py** — WebSocket client for the server. `converse()` opens `/converse`, streams mic PCM up on a send thread, and receives four kinds of frames back: `transcript` (text), `response` (text), PCM binary frames, and `done` (with `go_to_sleep` / `power_down` flags). PCM bytes are drained into `pcm_q` at network speed by the recv loop; a separate `_playback_loop` thread pulls from the queue and writes to PyAudio at real-time speed. This decoupling keeps the WebSocket lifetime ≈ "Gemini+TTS production time" regardless of reply length, avoiding Cloud Run / LB idle timeouts during playback. `speak()` is a simpler HTTP POST to `/speak` used only for the wakeup line.

**bear_state.py** — paw-button state machine (RUNNING/PAUSING/PAUSED/UNPAUSING/TERMINATING). GPIO 2 via gpiozero with `hold_time=1.0` and `bounce_time=0.05`, two handlers: short-press (`when_released`, < 1 s) is the **engage** action — wake from PAUSED, or `client.barge_in()` mid-utterance, or no-op if the bear is already listening; long-press (`when_held`, ≥ 1 s) is **sleep**, equivalent to the voice command "Zaby go to sleep". Different beep tones distinguish short vs long press. Volume is forced to 90% on every short press (hardware drifts). Barge-in does NOT change state — the main loop just re-enters `converse()` with no wakeup line, so the user hears beep → silence → ready to talk.

**bear_animatronics.py** — streaming PCM consumer: `start_audio(rate)`, `feed_audio(pcm)`, `end_audio()`. Computes RMS envelope at 200 Hz in `feed_audio`; a dedicated mouth-motor thread converts RMS → pulse durations (0 / 0.15s / 0.25s). Neck motor runs continuously while audio is streaming.

### Server side (`cloud_run/`)

**server.py** — FastAPI app with `/healthz`, `/speak` (text → PCM bytes), `/converse` (WebSocket), `/wakeup` (plain-text wakeup phrase, fetched by the Pi at boot — unauth so the Pi can call it), and `/memory` (HTML + form POSTs, HTTP basic auth). `/converse` runs STT in a thread while draining incoming audio frames, then streams Gemini tokens through a `SentenceBuffer` so each completed sentence hits TTS before the rest of the reply is generated. First sentence's PCM usually reaches the Pi while later sentences are still synthesizing. A single global `AIAgent` is held at module level — no per-client_id keying.

**ai_agent.py** — Gemini 3 Flash (`gemini-3-flash-preview`) with explicit history management. `interact_stream()` yields text chunks and drives the manual function-call loop: stream → collect any `function_call` parts → execute → append `function_response` → stream again. Automatic function calling is disabled (`AutomaticFunctionCallingConfig(disable=True)`) because automatic FC + streaming hangs (googleapis/python-genai#331). The system prompt + memory text are rebuilt fresh into `_build_config()` on every turn so live edits to `INSTRUCTIONS.md` and `MEMORY.md` take effect immediately. Tools: `reset_conversation`, `get_the_time` (uses `self.tz` from the WS `?tz=` param), `get_battery_voltage`, `go_to_sleep`, `power_down`, `save_memory`, `search_memory`. `GEMINI_API_KEY` is read from the environment.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daveyburke/Zaby](https://github.com/daveyburke/Zaby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
