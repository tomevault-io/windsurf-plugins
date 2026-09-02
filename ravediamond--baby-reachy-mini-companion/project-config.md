---
trigger: always_on
description: A **baby nursery companion** built on the Reachy Mini robot by Pollen Robotics. The project is a fully local AI stack — 7+ models orchestrated on-device with zero cloud dependency. It's entered in the **NVIDIA/HuggingFace GTC Golden Ticket contest** in two categories: **Reachy Mini usage** and **Ollama**.
---

# Baby Reachy-Mini Companion - Project Context

## What This Is

A **baby nursery companion** built on the Reachy Mini robot by Pollen Robotics. The project is a fully local AI stack — 7+ models orchestrated on-device with zero cloud dependency. It's entered in the **NVIDIA/HuggingFace GTC Golden Ticket contest** in two categories: **Reachy Mini usage** and **Ollama**.

The developer is a new dad who built this for his actual baby. His broader goal is to transition into robotics focused on helping people (inspired by Enchanted Tools' hospital companion robots). This is his first step toward accessible assistive robotics.

## Contest Context

- **Contest**: NVIDIA GTC Golden Ticket — 9 winners, each in a category
- **Categories entered**: Reachy Mini usage, Ollama
- **Key differentiators vs other contestants**:
  - The ONLY submission running fully local (no cloud APIs)
  - The ONLY one deploying on NVIDIA Jetson Orin
  - The ONLY one using local SLMs instead of cloud LLMs
  - Was #1 in community rankings
- **Judging criteria** (equal weight, 1-10 each): Technical Innovation, Effective Use of Technology, Potential Impact, Presentation Quality

## Architecture

### Processing Pipeline
```
Audio → Silero VAD → Faster-Whisper STT → Ollama LLM (with tool calling) → Kokoro TTS → Speaker
                                              ↓
                              Tool dispatch (16+ tools)
                              ↓           ↓           ↓
                          Camera/VLM   Signal alerts   Robot motion
```

### Autonomous Detection Loops (run in background)
- **Audio classifier** (YAMNet): Detects baby cries → sends Signal photo alert directly (bypasses LLM) + injects system event into LLM → LLM calls soothe_baby
- **Danger detector** (YOLO): Scans camera every 2s for hazardous objects (scissors, knives, forks) → sends Signal photo alert directly (bypasses LLM) + injects system event → LLM speaks warning
- **Signal poller**: Receives remote text messages → processed through LLM → responds via Signal

**Critical design rule**: Safety-critical notifications (cry alerts, danger alerts) are sent directly in handler code via `_send_cry_photo_alert()` / `_send_danger_photo_alert()`. They do NOT depend on the LLM calling tools — SLMs (3B–4B) can't reliably chain 3+ sequential tool calls.

### Movement System
- 100 Hz control loop with monotonic clock phase alignment
- Primary moves (dances, emotions) are mutually exclusive
- Secondary moves (speech sway via HeadWobbler) are additive offsets
- HeadWobbler: converts TTS audio stream into head movement for natural speech appearance

## Repository Structure

```
src/reachy_mini_conversation_app/
├── main.py                  # Entrypoint, wires all components together
├── console.py               # Headless mode with settings dashboard (FastAPI)
├── config.py                # Configuration from .env, includes feature flags
├── moves.py                 # 100Hz movement control loop
├── camera_worker.py         # 30Hz camera polling with frame buffering
├── prompts.py               # Dynamic prompt loading from profiles
├── local/
│   ├── handler.py           # Core pipeline: VAD→STT→LLM→TTS + system events
│   ├── llm.py               # OpenAI-compatible LLM client with streaming + tools
│   ├── stt.py               # Faster-Whisper wrapper
│   ├── tts.py               # Kokoro ONNX wrapper
│   └── vad.py               # Silero VAD wrapper
├── tools/
│   ├── core_tools.py        # Tool base class, registry, ToolDependencies dataclass
│   ├── camera.py            # VLM visual question answering
│   ├── soothe_baby.py       # Rocking motions + lullaby
│   ├── check_baby_crying.py # Query audio classifier status
│   ├── check_danger.py      # Query YOLO danger detector status
│   ├── send_signal.py       # Send text via Signal
│   ├── send_signal_photo.py # Capture frame + send via Signal
│   ├── dance.py             # Movement primitives
│   ├── story_time.py        # Children's story narration
│   ├── move_head.py         # Direct head control
│   └── ...
├── vision/
│   ├── processors.py        # VisionProcessor (API-based VLM) + VisionManager
│   └── danger_detector.py   # YOLO object detection for baby safety
├── audio/
│   ├── classifier.py        # YAMNet ONNX audio event classifier
│   ├── head_wobbler.py      # Audio-reactive head movement during speech
│   └── speech_tapper.py     # Loudness/VAD processing
├── input/
│   └── signal_interface.py  # Signal-CLI bridge
├── profiles/
│   └── default/
│       ├── instructions.txt # System prompt
│       └── tools.txt        # Enabled tools list
├── static/                  # Settings dashboard (HTML/CSS/JS)
│   ├── index.html           # Dashboard with LLM settings + feature toggles
│   ├── style.css
│   └── main.js
└── images/                  # Avatar images for Gradio chatbot
```

## Key Design Patterns

### System Events (autonomous action pattern)
Used by both audio classifier and danger detector:
1. Background task detects something (cry, dangerous object)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ravediamond/baby-reachy-mini-companion](https://github.com/ravediamond/baby-reachy-mini-companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
