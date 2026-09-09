---
trigger: always_on
description: Work as a senior software engineer on Nispa Voiceover, a local-first desktop application for subtitle voiceover and untimed multi-speaker dialogue generation.
---

# AGENTS.md — Engineering instructions for Nispa Voiceover

## Mission

Work as a senior software engineer on Nispa Voiceover, a local-first desktop application for subtitle voiceover and untimed multi-speaker dialogue generation.

The immediate product goal is to integrate OmniVoice as an additional local TTS provider and refactor the current two-provider implementation into a maintainable, data-driven architecture. Deliver working product functionality, not demos, placeholders, or model-specific branches scattered through the codebase.

The primary usage is English dialogue with UK-accented cloned voices. Italian and other languages remain supported but are not the main optimisation target.

## Source of truth and task workflow

Before changing code:

1. Read this file completely.
2. Read `TASKS.md`, `PLANNING.md`, and the files directly involved in the current task.
3. Inspect `git status --short` and preserve all unrelated user changes.
4. Verify claims against the current code, installed environment, and tests. Documentation may lag behind implementation; do not treat version numbers or architecture descriptions as authoritative without checking.
5. Identify the current phase in `TASKS.md` and work only on a coherent, testable slice.

After completing a slice:

1. Run focused tests for the changed area.
2. Run the broader regression suite appropriate to the risk.
3. Update `TASKS.md` checkboxes only for work actually implemented and verified.
4. Update documentation when public behaviour, installation, configuration, API payloads, or limitations change.
5. Report what changed, what was verified, and any remaining risk or decision.

Do not mark a task complete based only on mocked tests when the task explicitly requires a real model, GPU, installer, or offline smoke test.

## Non-negotiable product constraints

- TTS inference is local. Do not add cloud TTS APIs or remote fallbacks.
- Voice references, transcripts, embeddings, acoustic tokens, cached prompts, generated segments, and outputs are biometric or sensitive data. They must remain local and must not appear in telemetry, remote requests, fixtures, Git history, or verbose logs.
- Model download is an explicit installation action. Synthesis must never trigger an implicit download.
- Runtime must support strict offline operation after models and dependencies are installed.
- OmniVoice is an additional provider, regardless of whether it outperforms Qwen in every benchmark. Benchmarks determine recommendations and presets, not whether the provider exists.
- OmniVoice v1 is a per-utterance provider in the existing Script Mode. Do not claim or simulate native multi-speaker generation.
- Existing Qwen and VibeVoice workflows, archived jobs, voice files, and settings must remain backward compatible unless a migration is deliberately designed and tested.
## Current environment assumptions

Treat these as important project constraints, but verify the installed environment before changing dependencies:

- Primary development machine: NVIDIA RTX 5070 Ti Laptop, Blackwell `sm_120`, 16 GB VRAM.
- Current CUDA target: CUDA 13.2.
- Current PyTorch target: `2.10.0+cu130`, installed from the CUDA 13.0 PyTorch index.
- Flash Attention may be installed with `pip install flash-attn --no-build-isolation`.
- Do not switch to `cu124`, generic stable wheels, or nightly wheels just because an upstream README uses them. Blackwell support and the installed application environment take precedence.

Useful existing project patterns:

- Launch from `start.bat` / `start.sh`.
- Install through `install.bat` / `install.sh`; model downloads go through `backend/scripts/download_model.py`.
- Backend stack: FastAPI, SQLite, PyTorch, local model inference.
- Frontend stack: React, Vite, TypeScript, Tailwind.
- Keep TTS model loading lazy. Application startup must not load model weights.
- Save generated segments as WAV under `data/audio-rendering/` and final outputs under `data/outputs/`.
- Prefer `soundfile.write()` for generated WAV bytes. Do not reintroduce known `torchaudio.save()` / TorchCodec issues in Qwen paths.
- Use `asyncio.to_thread()` or a managed worker for blocking TTS work.

Known legacy risks to re-check before touching the area:

- `backend/core/tts_provider.py` currently routes providers through hard-coded pools and model-name heuristics.
- `backend/api/routers/voices.py` currently derives model/provider behaviour from discovered model folders and name checks.
- `backend/api/routers/tasks.py` contains script/dialogue orchestration, batching, cancellation, SSE progress, and speaker limits.
- `backend/api/routers/translation.py` may contain older Transformers argument usage; verify before editing translation code.
- `backend/db/database.py` may still have SQLite connection lifecycle issues; fix only if they block the current slice or are already in touched code.
- `backend/main.py` may still use deprecated FastAPI startup events; do not fold that cleanup into OmniVoice work unless it becomes necessary.

## Definition of professional implementation

### Build features, not patches


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nispa/nispa-vibevoice-studio](https://github.com/nispa/nispa-vibevoice-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
