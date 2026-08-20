---
trigger: always_on
description: HDMI passthrough with real-time ML-based ad detection and blocking using dual NPUs + CPU ASR:
---

# Minus - Development Notes

## Overview

HDMI passthrough with real-time ML-based ad detection and blocking using dual NPUs + CPU ASR:
- **PaddleOCR** on RK3588 NPU (~400ms per frame, 1.0s timeout)
- **LFM2.5-VL-450M (ft-v2-fused-v2)** on Axera LLM 8850 NPU — **prefill-only on 16 fused decoder layers, ~0.37s per frame deterministic** (1.5s soft / 2s hard timeout). Replaced FastVLM-0.5B iter4 (May 2026): 97.0% holdout accuracy / 99.2% non-ad-recall vs iter4's 94.75% / 95.25%, structurally simpler (no KV cache, no autoregressive decode for `detect_ad` OR autonomous-mode `query_image`, no ml_dtypes bfloat16 ceremony). Both inference paths share one model — no FastVLM dependency anymore. See *FastVLM iter4 → LFM2.5-VL Migration* under Known Issues.
- **Moonshine tiny-en (ONNX) ASR** on 3 pinned CPU cores (~1.6s per **2s** audio window, max <2s even on dense continuous speech). Runs in a multiprocessing worker subprocess (mirrors OCR/VLM worker pattern) for hard-timeout safety. **CONFIRM-ONLY** audio signal on top of OCR+VLM: decorates the block label (`+asr`) and does a gated mid-block rescue, but **never suppresses a block at start** (the old VETO was removed in 2026-05 — it was killing real ads VLM was sure about). Never fires blocking alone. Engine-selectable via `MINUS_ASR_ENGINE` (faster-whisper fallback for cool/idle hosts; on the thermally-throttled production box faster-whisper's fixed 30s encoder is ~3.3-5s/window — too slow, hence Moonshine which processes audio proportionally). Was whisper.cpp → faster-whisper → Moonshine. See [docs/ASR.md](docs/ASR.md) and *Moonshine ASR migration + decision-engine retune* under Known Issues.
- **Spanish vocabulary practice** during ad blocks!

## Documentation

| Document | Description |
|----------|-------------|
| [docs/FEATURES.md](docs/FEATURES.md) | Complete feature list and capabilities |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | System architecture and data flow |
| [docs/AESTHETICS.md](docs/AESTHETICS.md) | Visual design guide for UI/overlays |
| [MARISOL.md](MARISOL.md) | AI agent context guide |
| [docs/DEBUG_GLITCHES.md](docs/DEBUG_GLITCHES.md) | Video glitch debugging notes |
| [docs/FPS_DEBUGGING.md](docs/FPS_DEBUGGING.md) | FPS tracking and optimization |
| [docs/AUDIO.md](docs/AUDIO.md) | Audio passthrough documentation |
| [docs/ASR.md](docs/ASR.md) | Moonshine (ONNX) ASR — audio-based ad CONFIRM signal on top of OCR+VLM (worker process, 2s window, veto removed 2026-05) |
| [docs/VLM_NPU_DEGRADATION.md](docs/VLM_NPU_DEGRADATION.md) | Investigation of "NPU degradation" — root cause is per-image output-length variance; fix is `max_new_tokens` cap |
| [docs/IR_TRANSMITTER.md](docs/IR_TRANSMITTER.md) | IR transmitter for the REI 8K HDMI switch (PWM3 on pin 38) — wiring, NEC codes, API, troubleshooting |
| [docs/IR_RECEIVER.md](docs/IR_RECEIVER.md) | IR receiver eval on pin 3 (`gpiochip4 11`) — bench-tested decode of NEC remotes, gotchas, sketch for a future `IRReceiver` module |
| [docs/STATUS_LEDS.md](docs/STATUS_LEDS.md) | WS2812B status strip on SPI0 MOSI (pin 19) — wiring, state catalogue, API, encoding rationale |

## Visual Design

See **[docs/AESTHETICS.md](docs/AESTHETICS.md)** for the complete visual design guide including:
- Color palette (black background, matrix green, danger red, purple accents)
- Typography (VT323 for display, IBM Plex Mono for body, DejaVu for TV overlays)
- Component styling and animations
- TV overlay layout specifications

## Architecture

```
┌──────────────┐     ┌────────────────────┐     ┌─────────────────────┐
│   HDMI-RX    │────▶│     ustreamer      │────▶│  GStreamer Pipeline │
│ /dev/video0  │     │ (MJPEG encoding)   │     │  (queue + kmssink)  │
│  4K@30fps    │     │                    │     │                     │
│              │     │   :9090/stream     │     │                     │
│              │     │   :9090/snapshot   │     │                     │
└──────────────┘     └────────┬───────────┘     └─────────────────────┘
                              │
                              ▼ HTTP snapshot (~150ms, non-blocking)
              ┌───────────────┴───────────────┐
              │                               │
     ┌────────┴────────┐           ┌──────────┴──────────┐
     │   OCR Worker    │           │    VLM Worker       │
     │  ┌───────────┐  │           │  ┌───────────────┐  │
     │  │ PaddleOCR │  │           │  │  LFM2.5-VL    │  │
     │  │ RK3588 NPU│  │           │  │ Axera LLM 8850│  │
     │  │ ~400ms    │  │           │  │ ~0.37s        │  │
     │  └───────────┘  │           │  └───────────────┘  │
     └────────┬────────┘           └──────────┬──────────┘
              │                               │
              └───────────────┬───────────────┘
                              │
                     ┌────────┴────────┐
                     │ Blocking Mode   │
                     │ (ustreamer API) │
                     └─────────────────┘
```

**Key Architecture Points:**
- Simple GStreamer pipeline with `queue max-size-buffers=3 leaky=downstream`
- All blocking overlay rendering done in ustreamer's MPP encoder at 60fps
- No X11 required - uses DRM/KMS directly via kmssink

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [garagehq/Minus-streaming-stick](https://github.com/garagehq/Minus-streaming-stick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
