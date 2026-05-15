---
trigger: always_on
description: VoxTerm is a local, offline voice transcription TUI for macOS Apple Silicon. It captures mic + system audio, transcribes speech in real-time, identifies speakers, and remembers voices across sessions.
---

# CLAUDE.md — VoxTerm Agent Guide

## What is this project?

VoxTerm is a local, offline voice transcription TUI for macOS Apple Silicon. It captures mic + system audio, transcribes speech in real-time, identifies speakers, and remembers voices across sessions.

**Stack**: MLX (Qwen3-ASR transcription on Metal GPU) · 3D-Speaker ERes2Net (512-dim speaker embeddings via ONNX) · Silero VAD (ONNX, speech detection) · Textual (TUI) · SQLite (speaker profiles) · sounddevice (mic) · Swift/ScreenCaptureKit (system audio) · 3D-Speaker LID (language identification)

## Architecture

```
MAIN PROCESS
├─ Main thread (Textual event loop)
│  ├─ 15fps audio timer: reads mic + system audio queues
│  ├─ Silero VAD (ONNX, no PyTorch): speech/silence detection per chunk
│  ├─ UI rendering, keybindings (R/T/P/M/L/S/C/D/Q)
│  └─ SQLite reads for profile display
│
├─ Worker thread (@work(thread=True), group="transcription")
│  ├─ MLX transcription (Qwen3-ASR or Whisper)
│  ├─ 3D-Speaker diarization (ONNX, in-process — no subprocess needed)
│  │  ├─ ERes2Net-large (512-dim embeddings, best accuracy: 0.52% EER)
│  │  ├─ Pure-numpy Fbank features (no PyTorch/torchaudio)
│  │  └─ Online cosine clustering with spectral re-clustering
│  ├─ Language identification (3D-Speaker LID, ONNX)
│  ├─ Cross-session speaker matching (SQLite writes)
│  └─ call_from_thread() → UI updates
│
SUBPROCESSES (fallback only — not used when ONNX models available)
├─ Diarizer subprocess (PyTorch/speakerlab)
│  ├─ Loads 3D-Speaker model via speakerlab (fallback if ONNX unavailable)
│  ├─ Receives audio over pipe, returns speaker ID + embedding
│  ├─ Owns all session state (centroids, names, embeddings)
│  └─ Auto-restarts on crash; falls back to in-process if repeated failures
│
└─ System audio subprocess (Swift/ScreenCaptureKit)
   ├─ Compiled on first use from _macos_sck.swift
   ├─ Streams raw PCM over stdout pipe
   └─ For Bluetooth: routes through BlackHole virtual device
```

**Why ONNX?** The primary speaker embedding model (3D-Speaker ERes2Net) is exported to ONNX and runs via onnxruntime in the main process — no PyTorch needed, no subprocess needed. This eliminates IPC overhead and crash recovery complexity. The PyTorch subprocess path is kept as a fallback for when ONNX models aren't available.

**Legacy process isolation**: MLX (Metal GPU) and PyTorch (CPU) have C++ runtimes that conflict when loaded in the same process. The fallback diarizer subprocess prevents this, but is no longer needed with the ONNX backend.

## File map

| File | Description |
|------|-------------|
| `app.py` | Main Textual app — audio loop, transcription pipeline, session management, modals |
| `config.py` | Constants: sample rate, models, colors, paths, thresholds |
| `cyberpunk.tcss` | Textual CSS theme |
| `diagnostics.py` | Crash reporting: faulthandler, signal handlers, crash dumps, log rotation |
| `audio/capture.py` | Mic input via sounddevice callback → queue |
| `audio/system_capture.py` | System audio via Swift subprocess + pipe reader threads |
| `audio/vad.py` | Silero VAD wrapper (ONNX, no PyTorch) — neural speech/silence detection |
| `audio/buffer.py` | Thread-safe audio accumulator (append/get_and_clear) |
| `audio/platform.py` | macOS platform detection (Bluetooth, output device info) |
| `audio/blackhole.py` | BlackHole virtual device integration for Bluetooth routing |
| `audio/_macos_sck.swift` | ScreenCaptureKit Swift helper source |
| `audio/_macos_aggregate.swift` | Multi-output device Swift helper source |
| `transcriber/engine.py` | Qwen3-ASR (primary) + mlx-whisper (fallback), hallucination filter, dedup |
| `diarization/fbank.py` | Pure-numpy Mel filterbank (Kaldi-compatible, no PyTorch) |
| `diarization/onnx_embedder.py` | ONNX-based speaker embedding extraction (3D-Speaker models) |
| `diarization/campplus.py` | CAM++ model architecture (legacy, vendored from WeSpeaker) |
| `diarization/cluster.py` | 3D-Speaker clustering algorithms: spectral (p-value pruning), AHC, auto-select |
| `diarization/engine.py` | Online speaker clustering with ONNX/PyTorch backend dispatch |
| `diarization/proxy.py` | DiarizationProxy — direct (ONNX), subprocess, or inprocess modes |
| `diarization/subprocess_worker.py` | Subprocess entry point: loads model, read-process-write loop |
| `diarization/ipc.py` | Binary IPC protocol for main↔subprocess communication |
| `lid/engine.py` | Language identification using 3D-Speaker LID models (ONNX) |
| `scripts/export_onnx.py` | Export 3D-Speaker models to ONNX format |
| `speakers/models.py` | SpeakerProfile, SpeakerMeta dataclasses, multi-centroid matching |
| `speakers/store.py` | SQLite persistence, cross-session matching, backup/restore |
| `widgets/waveform.py` | FFT pixel-shader oscilloscope with pitch-mapped color |
| `widgets/transcript.py` | RichLog transcript with speaker labels + confidence indicators |
| `widgets/header.py` | Recording indicator header bar |
| `widgets/tag_screen.py` | Speaker tagging modal (T key) |
| `widgets/profile_screen.py` | Speaker profile management modal (P key) |

## Data and debug paths


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmarzzz/VoxTerm](https://github.com/dmarzzz/VoxTerm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
