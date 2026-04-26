---
trigger: always_on
description: An AI-powered Reachy Mini robot demo running on Jetson Orin. The robot ("The Witness") observes people, holds conversations, controls desktop apps via gestures, and reacts with emotions, lighting, and head/body movement.
---

# Jarvis - Reachy Mini Hackathon Robot

An AI-powered Reachy Mini robot demo running on Jetson Orin. The robot ("The Witness") observes people, holds conversations, controls desktop apps via gestures, and reacts with emotions, lighting, and head/body movement.

## Platform

- **Hardware:** Jetson Orin Nano, Reachy Mini robot, ZED 2 stereo camera, Hollyland wireless mic, Bluetooth speaker
- **OS:** JetPack 6 (L4T R36.4.3), CUDA 12.6, TensorRT 10.3
- **Python env:** Conda `trt_pose` env has torch 2.5 (CUDA), torchvision 0.19, OpenCV 4.8, ultralytics
- **Run with:** `/home/orin/miniconda3/envs/trt_pose/bin/python` (needs `LD_LIBRARY_PATH` for cusparselt, see below)
- **Docker alternative:** `run_docker.sh` runs `witness:latest` container with NVIDIA runtime, host network, device access

```bash
# Required env for running outside Docker:
export LD_LIBRARY_PATH="/home/orin/miniconda3/envs/trt_pose/lib/python3.10/site-packages/nvidia/cusparselt/lib:${LD_LIBRARY_PATH}"
```

## Architecture Overview

```
                    main.py (orchestrator)
                    State machine: IDLE → AMBIENT → ENGAGED
                    4 threads: listen, brain, output, gesture
                         │
    ┌────────────────────┼────────────────────────┐
    │                    │                         │
 INPUT               COGNITION                 OUTPUT
 ─────               ─────────                 ──────
 listen.py            brain.py                  speak.py
  (mic→ASR)            (LLM orchestration)       (Piper TTS)
 vision.py            memory.py                 reachy_bridge.py
  (camera+VLM)         (person facts store)       (head/body/antenna)
 faces.py             openclaw_bridge.py        hue.py
  (YuNet+SFace)        (desktop commands)         (Philips Hue lights)
 gestures.py          llm_proxy.py              actions.py
  (hand pose)          (Agora LLM proxy)          (gesture→keyboard)
 person_tracker.py                              dashboard.py
  (YOLOv8-pose+ReID)                              (MJPEG web UI)
```

## main.py - The Orchestrator

Entry point. Runs 4 daemon threads coordinated through a shared `State` object and queues.

### State Machine
- **IDLE** - No faces detected. Waiting.
- **AMBIENT** - Face(s) present. Robot observes, greets arrivals (after 3s confirmation), makes periodic ambient comments.
- **ENGAGED** - Active conversation locked to one person (closest to center). Speech routed to LLM, responses spoken via TTS.

### Threads
1. **listen_loop** - Mic → faster-whisper (local) or OpenAI Whisper API (cloud) → `audio_queue`. Ignores speech while TTS is playing (prevents feedback loop).
2. **brain_loop** - Polls `audio_queue` for speech. Grabs camera frame, runs face events (arrivals/departures), gets scene description from VLM, builds memory context, calls LLM, puts response on `response_queue`. Also handles: goodbye detection, OpenClaw command routing, silence timeout (→ ambient), face departure (→ ambient).
3. **output_loop** - Reads `response_queue`. Executes robot actions (emotion, head, antenna). Sets Hue light emotion. Speaks response via TTS (blocking).
4. **gesture_loop** - Independent. Camera → hand pose → gesture classification → xdotool desktop actions.

### Key CLI Flags
```
--no-vlm          Skip vision-language model
--no-tts          Print instead of speak
--no-robot        Skip robot commands (default: True)
--no-listen       Skip mic input
--no-gestures     Skip hand gesture recognition
--no-openclaw     Skip desktop command routing
--no-hue          Skip Philips Hue lights
--agora           Use Agora cloud ASR+LLM+TTS (replaces local listen/output)
--llm-backend     local | openai | openrouter
--asr-backend     local | cloud
--vlm-backend     transformers | llama_cpp
--ambient-interval   Seconds between ambient lines (default: 30)
--silence-timeout    Seconds before engaged → ambient (default: 30)
--departure-buffer   Seconds face must be gone before counting as departed (default: 10)
```

### Two Modes of Operation
1. **Local mode** (default): listen_loop + brain_loop + output_loop + gesture_loop + dashboard on :8080
2. **Agora mode** (`--agora`): Agora web server on :8080, browser handles RTC voice. Agent manages cloud ASR+LLM+TTS. Brain loop still runs for face detection. Optional MCP server for OpenClaw tool calling.

## Pipeline Modules

### listen.py - Audio Input / ASR
- Captures from Hollyland mic via `sounddevice` (48kHz → resample to 16kHz)
- VAD for speech/silence detection
- Backends: `faster_whisper` (local, GPU) or OpenAI Whisper API
- Output: `text_queue` of (text, timestamp) tuples

### speak.py - TTS Output
- `SpeakPipeline`: Piper TTS CLI → WAV → `paplay` (PulseAudio, routes to Bluetooth speaker)
- `DummySpeakPipeline`: prints text (for `--no-tts`)
- `say_blocking(text)` blocks until audio finishes playing

### brain.py - LLM Orchestration
- Maintains per-face conversation history
- Loads system prompt from `config/system_prompt.txt`
- `engage(text, scene_desc, face_id, memory_context)` → JSON response
- `greet(face_id, person_info, scene_desc)` → greeting response

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/irenegracekp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
