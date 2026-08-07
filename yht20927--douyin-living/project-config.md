---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Douyin (Chinese TikTok) Live AI Clipper — records live streams and automatically detects + clips "highlight moments" using multimodal signal fusion (danmaku, audio, visual, ASR).

**Pipeline**: Record → Danmaku + FLV → Audio extract → ASR → Signal extraction (3 parallel) → Scoring → FFmpeg clipping

## Commands

```bash
# Record a live room (danmaku + video)
python -m src.controller 300294032039

# Record + auto-clip when done
python -m src.controller 300294032039 --clip

# Clip from existing recordings
python -m src.controller 300294032039 --clip-only --profile game --sensitivity 2.0

# Standalone tools
python extractAudio.py video.flv           # → .aac
python asr.py audio.aac --model tiny       # → _asr.json + _asr.srt

# Install dependencies
pip install -r requirements.txt
npm --prefix scripts install               # Node.js signing (scripts/dyAb.js)
```

## Architecture

### Recording Layer (async)
- **`src/controller.py`** — CLI entry point + orchestrator. Three modes: record, record+clip, clip-only. Coordinates all phases.
- **`src/danmakuWs.py`** — WebSocket client (non-async, runs in thread via `ThreadWs`). Receives danmaku/chat/gifts/likes as Protobuf, writes JSONL.
- **`src/flvRecorder.py`** — FLV video stream recorder via aiohttp chunked download.
- **`src/roomApi.py`** — Douyin live room API. Scrapes SSR HTML for room info + signed FLV URLs. Uses Protobuf for webcast detail.
- **`src/auth.py`** — Cookie-based auth. Reads `DY_COOKIES`/`DY_LIVE_COOKIES` from `.env`.
- **`src/signer.py`** — Generates a_bogus/X-Bogus/msToken via PyExecJS executing `scripts/dyAb.js`.
- **`src/params.py`** — URL parameter builder with `.withABogus()` chaining.

### AI Clipping Layer
All features are **per-second integer arrays** (sampleRate=1). The three signal extractors run **in parallel** via `ThreadPoolExecutor`.

- **`src/signalAudio.py`** — librosa (RMS, spectral centroid, MFCC, ZCR) + panns-inference (laughter, applause, music). Uses `_fastSentiment` tri-state caching (True/False/None) to avoid repeated HF pipeline loads.
- **`src/signalText.py`** — jieba tokenization + FastText fuzzy keyword matching (60% threshold) + HDBSCAN topic clustering + HF sentiment analysis. Slow models (FastText, text2vec, HF) load on-demand.
- **`src/signalVisual.py`** — PySceneDetect (content threshold=27.0) + OpenCV optical flow (Farneback, sampled every 5 frames at 320×240). Face detection placeholder for Phase 2.
- **`src/scorer.py`** — **Core differentiator**. Fuses ~15 signals via weighted profiles → EWMA → rolling Z-Score (60s window) → median filter → cross-correlation lag compensation → multi-scale peak detection (3s/10s/30s windows) → NMS → semantic boundary optimization → t-test quality filter. Outputs `time_table.json` + `scores.json`.
- **`src/clipper.py`** — FFmpeg wrapper for lossless clipping (`-c copy`), SRT subtitle extraction/burn-in, thumbnail extraction. External dep: ffmpeg binary.

### Data Flow
```
data/{roomId}/
├── {roomId}_{ts}.flv          ← raw video
├── {roomId}_{ts}.aac          ← lossless audio extract
├── {roomId}_danmaku.jsonl     ← chat/gifts/likes
├── {roomId}_{ts}_asr.json     ← ASR segments + speakers
├── {roomId}_{ts}_asr.srt      ← SRT subtitles
├── audio_features.json        ← signalAudio output
├── text_features.json         ← signalText output
├── visual_features.json       ← signalVisual output
├── time_table.json            ← scorer clip candidates
├── scores.json                ← per-second score timeline
└── highlights/                ← final clips (.mp4 + .srt + .jpg)
```

### Scoring Profiles
Weight profiles in `config/scorer.yaml` (profiles section): `default`, `game`, `shopping`, `talent`. Each assigns weights to ~15 signals (dmDensity, asrKeyword, eventLaughter, rms, motion, etc.). Weights must sum ~1.0. Loaded via `src/config/settings.py`.

### Protobuf
- **`src/protobuf/Live_pb2.py`** — Generated from Douyin's LiveResponse proto. Used to decode webcast detail (cursor, internalExt) and WS message frames.
- **`src/protobuf/decoder.py`** — Decodes binary WS frames → list of message dicts. Includes `_encode_varint` with negative-value guard.

### Config System (new in Sprint 2)
- **`src/config/settings.py`** — Pydantic models for all settings, loaded from YAML with env-var override. Cached via `@lru_cache`. Use `load_settings()` to access, `reload_settings()` to force reload.
- **`config/scorer.yaml`** — Profiles, preprocessing (EWMA, Z-score), detection (windows, NMS, peak persistence), boundary optimization, t-test, lag compensation
- **`config/audio.yaml`** — Sample rate, librosa params, panns thresholds/batch sizes, min trigger gap
- **`config/text.yaml`** — Similarity/UTR thresholds, model paths, window sizes, clustering params
- **`config/visual.yaml`** — Scene detection threshold, optical flow params, face detection score
- **`config/recording.yaml`** — FLV rotation, backoff/retry, HTTP timeouts, WS ping interval
- **`config/pipeline.yaml`** — Default sensitivities, pool workers, ffmpeg timeouts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yht20927/douyin-living](https://github.com/Yht20927/douyin-living) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
