---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LiveTranslate is a real-time audio translation system for video players on Windows. It captures system audio via WASAPI loopback, runs speech recognition, and translates via LLM APIs, displaying results in a transparent overlay.

**Current phase**: Phase 0 Python prototype (Phase 1 will be a C++ DirectShow Audio Tap Filter).

## Running

```bash
# Must use the project venv (system Python lacks dependencies)
.venv/Scripts/python.exe main.py
```

Linter: `ruff` (installed globally). Run `python -m ruff check --select F,E,W --ignore E501,E402 *.py` to lint. E402 is intentionally ignored because `main.py` requires torch before PyQt6.

## Architecture

The pipeline runs in a background thread: **Audio Capture (32ms chunks) -> VAD -> ASR -> Translation (async) -> Overlay**

```
main.py (LiveTranslateApp)
  |-- model_manager.py     Centralized model detection, download, cache utils
  |-- audio_capture.py     WASAPI loopback via pyaudiowpatch, auto-reconnects on device change
  |-- vad_processor.py     Silero VAD / energy-based / disabled modes, progressive silence + backtrack split
  |-- asr_engine.py        faster-whisper (Whisper) backend
  |-- asr_sensevoice.py    FunASR SenseVoice backend (better for Japanese)
  |-- asr_funasr_nano.py   FunASR Nano backend
  |-- asr_qwen3.py         Qwen3-ASR backend (ONNX Encoder + GGUF Decoder)
  |-- qwen_asr_gguf/       Qwen3-ASR inference engine (from Qwen3-ASR-GGUF project)
  |-- translator.py        OpenAI-compatible API client, streaming, JSON schema, context history
  |-- subtitle_overlay.py  PyQt6 transparent overlay (2-row header: controls + model/lang combos)
  |-- subtitle_window.py   Standalone subtitle window for OBS capture (outlined text, animations)
  |-- subtitle_settings.py Subtitle window settings UI (grid layout, text line editor)
  |-- control_panel.py     Settings UI (7 tabs: VAD/ASR, Translation, Style, Subtitle, Benchmark, Cache, Changelog)
  |-- dialogs.py           Setup wizard, model download/load dialogs, ModelEditDialog
  |-- benchmark.py         Translation benchmark (BENCH_SENTENCES, run_benchmark())
  |-- log_window.py        Real-time log viewer
```

### Threading Model

- **Main thread**: Qt event loop (all UI)
- **Pipeline thread**: `_pipeline_loop` in `LiveTranslateApp` - reads audio, runs VAD/ASR/translation
- **ASR loading**: Background thread via `_switch_asr_engine` (heavy model load, ~3-8s)
- Cross-thread UI updates use **Qt signals** (e.g., `add_message_signal`, `update_translation_signal`)
- ASR readiness tracked by `_asr_ready` flag; pipeline drops segments while loading

### Configuration

- `config.yaml` - Base configuration (audio, ASR, translation, subtitle defaults)
- `user_settings.json` - Runtime settings persisted by control panel (models, VAD params, ASR engine choice, optional `cache_path`). Takes priority over config.yaml on load.

### Model Config

Each model in `user_settings.json` has: `name`, `api_base`, `api_key`, `model`, `proxy` ("none"/"system"/custom URL), and optional per-model flags:

- `no_system_role` (bool): Merges system prompt into user message for APIs that reject system role (e.g. Qwen-MT)
- `no_think` (bool, default True): Passes `extra_body={"enable_thinking": False}` to disable reasoning for thinking models (Qwen3 etc.)
- `streaming` (bool, default True): Per-model streaming toggle; streaming mode yields partial text via `translate_iter()` generator
- `json_response` (bool, default False): Uses `response_format={"type": "json_schema"}` with schema `{"t": "string"}` for structured output; mutually exclusive with streaming UI display
- `context_turns` (int, default 0): Number of recent (source, translation) pairs to include as multi-turn context in messages
- `input_price`/`output_price` (float, per 1M tokens): Token pricing for cost estimation displayed in MonitorBar
- Proxy handling: `proxy="none"` uses `httpx.Client(trust_env=False)` to bypass system proxy; `proxy="system"` uses default httpx behavior (env vars)
- Editing the active model in ModelEditDialog triggers `model_changed` signal to immediately apply changes

### Subtitle Window (subtitle_window.py)

Standalone transparent window for OBS capture, separate from the main overlay:
- `SubtitleWindow`: Frameless, transparent, middle-click draggable, auto-hides after timeout, default position (100, 100)
- `_SubtitleTextWidget`: QPainterPath-based outlined text rendering per line, with automatic word-wrap and entry/exit animations
- Text rendered to cached QPixmap; animations only blit the cached image (no per-frame path rendering)
- Each line has independent font, color, outline, alignment, animation settings
- Long text auto-wraps at punctuation/word boundaries, expanding widget height for multiple lines
- `desired_height()` returns height based on wrapped line count (never 0), preventing window collapse when empty
- Window height animates smoothly (150ms OutCubic) when wrap count changes, keeping vertical center stable
- Animation order: exit animation → height change → entry animation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheDeathDragon/LiveTranslate](https://github.com/TheDeathDragon/LiveTranslate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
