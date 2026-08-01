---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

easyVmaf is a Python CLI tool that wraps FFmpeg and FFprobe to compute VMAF video
quality scores. It handles the preprocessing that VMAF requires: deinterlacing,
scaling, frame rate normalization, and frame-accurate time synchronization between
reference and distorted video streams.

## Setup

```bash
pip install -e .          # from source (editable)
# or once published to PyPI:
pip install easyvmaf
```

FFmpeg >= 5.0 built with `--enable-libvmaf` must be on PATH, or override via env:

```bash
FFMPEG=/path/to/ffmpeg FFPROBE=/path/to/ffprobe python3 -m easyvmaf ...
```

## Running the Tool

```bash
# Installed CLI command
easyvmaf -d distorted.mp4 -r reference.mp4
easyvmaf -d distorted.mp4 -r reference.mp4 -sw 2      # with sync window
easyvmaf -d distorted.mp4 -r reference.mp4 --gpu      # GPU-accelerated (CUDA)
easyvmaf -d distorted.mp4 -r reference.mp4 -json      # structured JSON output
easyvmaf -d "folder/*.mp4" -r reference.mp4           # batch

# Module invocation (no install)
python3 -m easyvmaf -d distorted.mp4 -r reference.mp4
```

## Docker

```bash
# CPU build
docker build -t easyvmaf .
docker run --rm -v $(pwd)/video_samples:/videos easyvmaf \
  -d /videos/distorted.mp4 -r /videos/reference.mp4

# GPU build (requires CUDA 12.3, nvidia-container-toolkit on host)
docker build -f Dockerfile.cuda -t easyvmaf:cuda .
docker run --rm --gpus all -v $(pwd)/video_samples:/videos easyvmaf:cuda \
  -d /videos/distorted.mp4 -r /videos/reference.mp4 --gpu

# docker-compose
docker compose build
docker compose run easyvmaf -d /videos/dist.mp4 -r /videos/ref.mp4
```

---

## Three-Layer Architecture

Each layer must only talk to the layer directly below it.

```
easyvmaf/cli.py     ← Layer 3: CLI only (argparse, glob, JSON output, print results)
easyvmaf/vmaf.py    ← Layer 2: VMAF logic (scaling, deinterlace, sync, scoring)
easyvmaf/ffmpeg.py  ← Layer 1: FFmpeg/FFprobe subprocess wrappers
easyvmaf/config.py  ← binary path resolution (ffmpeg, ffprobe via shutil.which)
```

Supporting entry points:
- `easyvmaf/__init__.py` — public API surface
- `easyvmaf/__main__.py` — enables `python3 -m easyvmaf`

### Layer 1 — easyvmaf/ffmpeg.py
Thin subprocess wrappers around ffmpeg and ffprobe binaries.
- `FFprobe`: runs ffprobe, returns stream/frame/packet/format info as dicts
- `FFmpegQos`: builds and runs ffmpeg filter graph for PSNR and VMAF computation
- `inputFFmpeg`: manages per-input filter chains (scale, trim, fps, deinterlace, hwupload_cuda)
- `check_ffmpeg()`: probes FFmpeg version, built-in model availability, and `libvmaf_cuda` support
- `VMAF_MODELS`: structured dict defining HD and 4K model configurations
- `_build_model_string()`: builds the libvmaf `model=` parameter string

Must NOT contain any VMAF business logic or user-facing print statements.

### Layer 2 — easyvmaf/vmaf.py
VMAF computation orchestration.
- `video`: parses stream metadata via FFprobe (lazy loading), detects interlacing
- `vmaf`: auto-scaling, auto-deinterlace, parallel sync offset search, final VMAF scoring
- `UnsupportedFramerateError`: raised when no deinterlace filter covers the fps combination
- `FeatureConfig`: dataclass for building the libvmaf `feature=` parameter string

Must NOT contain CLI argument parsing or result formatting.

### Layer 3 — easyvmaf/cli.py
CLI entry point only. Argparse, glob pattern expansion for batch processing,
reading VMAF output files (json/xml/csv), printing or emitting structured JSON results.
- `-json` flag: emits NDJSON to stdout (one object per file in batch); logging goes to stderr
- `_build_result()`: constructs the result dict for JSON output and human-readable display
- `check_ffmpeg()` called at startup for version/model/CUDA validation

Must NOT contain FFmpeg filter logic or VMAF computation directly.

---

## FFprobe Call Map — Critical Reference

Understand this before touching easyvmaf/vmaf.py or easyvmaf/ffmpeg.py.

| Data               | Method            | Consumers in vmaf.py                          | Cost  |
|--------------------|-------------------|-----------------------------------------------|-------|
| `streamInfo`       | `getStreamInfo()` | `_autoScale` (width, height)                  | low   |
|                    |                   | `_autoDeinterlace` (r_frame_rate)             |       |
|                    |                   | `_deinterlaceFrame/Field` (r_frame_rate)      |       |
|                    |                   | `syncOffset` (r_frame_rate, width, height)    |       |
|                    |                   | `getVmaf` (r_frame_rate, width, height,       |       |
|                    |                   |   cambi feature string)                       |       |
|                    |                   | `getDuration` (primary: duration, start_time) |       |
| `formatInfo`       | `getFormatInfo()` | `getDuration` fallback only (KeyError path)   | low   |
| `framesInfo` /     | `getFramesInfo()` | `_autoDeinterlace` via `self.interlaced` only | HIGH  |
| `interlaced`       |                   | Skipped entirely when `manual_fps != 0`       |       |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gdavila/easyVmaf](https://github.com/gdavila/easyVmaf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
