---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Cursor, Codex, Aider, Gemini
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Codex, Aider, Gemini
CLI, Copilot) working in this repository. Humans should also read this —
it's the fastest way to get the lay of the land.

> **The two source-of-truth docs you should follow for the workflow are
> [`README.md`](README.md) (pipeline overview, commands, architecture)
> and [`docs/SETUP_GUIDE.md`](docs/SETUP_GUIDE.md) (zero-to-trained on
> a fresh GCP VM).** This file is a *navigation aid* — it points you at
> the right places and codifies conventions. It is not a replacement for
> the README.

## What this repo is

A finetuning workspace for **Moshi 7B** ([kyutai-labs/moshi](https://github.com/kyutai-labs/moshi))
and its NVIDIA variant **PersonaPlex** ([nvidia/personaplex-7b-v1](https://huggingface.co/nvidia/personaplex-7b-v1)).
Two domains have been used to validate the pipeline: a *companion* chat
finetune and a *pharma* patient-support finetune with mid-conversation
context injection (a "puppeteer" LLM splices facts into the model's text
stream while audio plays). Architectural quirks (frame layout,
`dep_q=16`, weight-loader hook chain, system-prompt prefix) are
described in [`docs/history/notes/DUPLEX_AND_FINETUNING_NOTES.md`](docs/history/notes/DUPLEX_AND_FINETUNING_NOTES.md)
— read that before changing anything in `moshi-finetune/finetune/` or
`personaplex/moshi/moshi/`.

## End-to-end workflow (and where each step is documented)

The full pipeline is **8 stages**, all documented in detail in
[`README.md`](README.md) under "Pipeline steps". Brief map:

| # | Stage | Script | Output | README §  |
|---|---|---|---|---|
| 1 | Generate synthetic dialogues (Claude API, LHS-sampled personas) | `pipeline/generate_dialogues_sync.py` | `<dataset>.jsonl` | 1 |
| 2 | Parse JSONL → "Speaker N:" plain-text scripts | `pipeline/parse_dialogues.py` | `scripts/<id>.txt` | 2 |
| 3 | Render scripts → mono audio (VibeVoice 7B TTS, multi-GPU) | `pipeline/generate_audio.py` | `mono_wav/<id>.wav` | 3 |
| 4 | WhisperX align + route to stereo channels | `pipeline/create_stereo.py` | `stereo_wav/<id>.{wav,json}` | 4 |
| 5 | Compute frame offsets for context injections | `pipeline/compute_injection_offsets.py` | annotates `stereo_wav/<id>.json` | 5 |
| 6 | Build train/eval manifest | `pipeline/create_manifest.py` | `dataset/{train,eval}.jsonl` | 6 |
| 7 | Finetune (FSDP + LoRA, multi-GPU torchrun) | `moshi-finetune` | `runs/<run-name>/` | 7 |
| 8 | Merge LoRA + serve via WebSocket | `pipeline/merge_lora.py`, `personaplex/moshi/moshi/server.py` | merged checkpoint, live server | 8 |

Eval (LLM judge over generated transcripts, A/B preference) and run
comparison are documented in README §"Evaluation".

For a fresh-machine setup (NVIDIA drivers, CUDA 12.8, Python envs,
HuggingFace cache, gated model downloads, training launch on a GCP VM),
follow [`docs/SETUP_GUIDE.md`](docs/SETUP_GUIDE.md). It walks the entire
journey from a blank Debian 12 box through a finished training run.

## Project map

```
configs/                        Training recipes (YAML). One file = one experiment.
data/                           (gitignored) Training audio + manifests. See "Data structure" below.
docs/
├── SETUP_GUIDE.md              Fresh-VM zero-to-trained walkthrough.
└── history/                    Frozen development artifacts (notes + wandb runs).
moshi-finetune/                 Vendored Kyutai finetune code (Apache-2.0). DO NOT pull from upstream.
personaplex/                    Vendored NVIDIA PersonaPlex code (MIT). DO NOT pull from upstream.
  moshi/                        Patched moshi runtime (dep_q=16, context injection).
  client/                       React/TS UI for serving + A/B testing.
pipeline/                       Top-level orchestration scripts (data prep, eval, manifest).
  utils/                        One-off / debugging utilities.
plans/                          (gitignored) Local planning scratch.
runs/                           (gitignored) Training outputs. See "Run structure" below.
VibeVoice/                      Vendored TTS used for synthetic data generation.
```

## Data structure (canonical example: `data/adhery-short/`)

A single dataset is a directory under `data/` with these subdirs, each
produced by a different pipeline stage:

```
data/<dataset>/
├── <dataset>.jsonl              Stage 1 output: source synthetic dialogues
├── scripts/<id>.txt             Stage 2: VibeVoice-format "Speaker 1:/Speaker 2:" plain text
├── speaker_samples/*.wav        Voice library (one wav per voice; deterministic per-dialogue assignment)
├── mono_wav/<id>.wav            Stage 3: VibeVoice-rendered single-channel audio (24 kHz)
├── stereo_wav/<id>.wav          Stage 4: 2-channel WAV (L=agent, R=user)
├── stereo_wav/<id>.json         Stage 4 sidecar: alignments + turns + prompts (see below)
└── dataset/
    ├── train.jsonl              Stage 6: training manifest
    └── eval.jsonl               Stage 6: held-out eval manifest
```

### Source dialogue row (`<dataset>.jsonl`, one row per dialogue)

```json
{
  "id": "adhv3-00000",
  "assistant_name": "Mia",
  "seed": {
    "patient_name": "...", "patient_age": 72, "drug": "Enzalutamide ...",
    "scenario_type": "edge-case", "tenor": "urgent ...",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tin-computer/personaplex-finetune](https://github.com/tin-computer/personaplex-finetune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
