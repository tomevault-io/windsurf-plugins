---
trigger: always_on
description: - Break all multi-step tasks into a numbered plan before starting
---

# StemForge — Claude Context

## Workflow
- Break all multi-step tasks into a numbered plan before starting
- After each major step, commit and push, then pause and report: what was done, what changed, what's next
- Wait for explicit "continue" or "proceed" confirmation before moving to the next step

## What this project is

AI-powered audio processing web application with six core pipelines:
- **Demucs** — source separation (vocals, drums, bass, other) — 4 models
- **BS-Roformer** — high-quality separation with 2-stem, 4-stem, and 6-stem (guitar + piano) models
- **BasicPitch** — polyphonic MIDI extraction from separated stems (instruments)
- **Vocal MIDI** — vocal pitch-to-MIDI via faster-whisper + PYIN pitch tracking
- **Stable Audio Open** — text-conditioned audio generation with optional audio and MIDI conditioning (Synth tab)
- **AceStep** — full song generation from style descriptions + lyrics (Compose tab, runs as subprocess)

Additional systems:
- **Enhance** — three-mode vocal enhancement tab:
  - **Clean Up** — UVR denoise, dereverb, debleed via vendored `python-audio-separator` fork (8 curated presets across Roformer/MDXC/VR architectures)
  - **Tune** — auto-tune via CREPE neural pitch detection (`torchcrepe`) + selectable resynthesis method: WORLD vocoder (`pyworld`, best on lossless audio) or STFT phase vocoder with formant preservation (`stftpitchshift`, better on compressed/MP3); scale snapping with correction strength and humanization controls
  - **Effects** — per-stem channel strip with 4 effect types (EQ, Compressor, Noise Gate, Stereo Width), each offering DSP and/or ML methods: 3-band parametric EQ (`scipy.signal`), DSP/LA-2A neural compressor (`vendor/micro_tcn`, Apache 2.0), DSP/Spectral (`torchgating`)/DeepFilterNet(disabled, numpy<2.0 conflict) noise gate, Mid/Side stereo width
- **Model registry** (`models/registry.py`) — frozen `ModelSpec` descriptors for all models; single source of truth for device rules, sample rates, capabilities, metadata, and pipeline defaults
- **Audio profiler** (`utils/audio_profile.py`) — spectral analysis that recommends the best engine/model for a given audio file
- **Mix engine** — multi-track mixer combining audio stems and MIDI-rendered tracks with per-track instrument, volume, and FLAC render
- **SFX Stem Builder** (`backend/api/sfx.py`) — DAW-style canvas for placing audio clips on a timeline, aligned to a reference stem, with per-clip fades and volume; renders to a single stem for the Mix engine

**Architecture**: FastAPI backend (`backend/`) + vanilla HTML/CSS/JS frontend (`frontend/`) + AceStep subprocess.
Run with `python run.py` → open `http://localhost:8765` in browser.
AceStep runs on port 8001 by default. Disable with `--no-acestep`.

---

## Current state

All pipelines and the full web UI are implemented:

- Demucs separation — 4 models (htdemucs, htdemucs_ft, mdx_extra, mdx_extra_q), CUDA fallback for MDX-Net
- BS-Roformer separation — 6 models including ViperX vocals (SDR 12.97), KJ vocals, ZFTurbo 4-stem, jarredou 6-stem
- Automatic engine/model recommendation from spectral audio analysis
- MIDI extraction — BasicPitch for instruments, faster-whisper + PYIN pitch for vocals
- MIDI preview — server-side FluidSynth render, streamed to browser via wavesurfer.js
- Mix tab — per-track volume controls, audio/MIDI source types, FLAC render, multi-track preview
- Stable Audio Open generation (Synth tab) — text + audio + MIDI conditioning, up to 600 s (chunked at 47 s), Vocal Preservation Mode
- SFX Stem Builder (Synth tab) — DAW timeline, clip placement with fades, align-to reference waveform, render canvas to Mix
- AceStep generation (Compose tab) — full song creation/rework, AI lyrics, 3-column UI, cross-tab integration, LoRA adapter management, project save/load, seed recall, dismissable result cards
- AceStep LoRA training (Compose tab Train mode) — upload audio, scan/label/preprocess pipeline, LoRA/LoKR fine-tuning with loss chart, snapshot management, adapter export
- Batch separation — multi-file upload, single-stem extraction across all files, Save All zip download
- Upload supports audio (WAV, FLAC, MP3, OGG, AIFF) and video (MP4, MKV, WEBM, AVI, MOV) — video audio extracted via FFmpeg
- Export panel — all pipeline outputs, 4 audio formats (wav/flac/mp3/ogg), zip download
- Waveform visualization via wavesurfer.js with global transport bar
- Deterministic uv environment, Python 3.11, CUDA 13.0 wheels
- macOS support via MPS acceleration (separate `pyproject.toml.MAC`)

---

## Project structure

```
StemForge/
├── run.py                          # Launcher: uvicorn + AceStep subprocess management
├── pyproject.toml
├── pyproject.toml.MAC              # macOS variant (MPS, no CUDA index)
│
├── Ace-Step-Wrangler/              # Git submodule (independently runnable)
│   ├── vendor/ACE-Step-1.5/        # Nested submodule — upstream AceStep
│   ├── backend/                    # Wrangler's standalone backend (reference)
│   ├── frontend/                   # Wrangler's standalone frontend (reference)
│   └── run.py                      # Wrangler's standalone launcher (unused in StemForge)
│
├── backend/
│   ├── __init__.py
│   ├── main.py                     # FastAPI app, router registration, static mount
│   ├── api/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tsondo/StemForge](https://github.com/tsondo/StemForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
