---
trigger: always_on
description: **Living dev contract for any agent (human, Claude, GPT, Gemini) working on strands-cosmos.**
---

# strands-cosmos — AGENTS.md

**Living dev contract for any agent (human, Claude, GPT, Gemini) working on strands-cosmos.**

---

## The 30-Second Pitch

**strands-cosmos = Full-lifecycle NVIDIA Cosmos toolkit for Strands Agents.**

- **Model Providers**: `CosmosModel` (text-only) + `CosmosVisionModel` (video+image+text) using Cosmos-Reason2 via HF Transformers
- **21 Tools**: Inference, video generation (Predict2.5), video-to-video (Transfer2.5), data curation (Xenna), post-training, distillation, quantization, edge deployment, evaluation
- **Edge-first**: Verified on Jetson AGX Thor (132GB), Orin, plus cloud GPUs
- **justfile as truth**: All pipeline commands flow through `just <recipe>`; tools are thin Python wrappers

---

## Core Principles

1. **RUN FIRST** — `pip install strands-cosmos` → `from strands_cosmos import CosmosVisionModel` works immediately.
2. **JUSTFILE IS TRUTH** — Every tool calls a `just` recipe. Change the pipeline? Edit the justfile.
3. **UPSTREAM UNTOUCHED** — Cosmos repos live alongside (`../cosmos-*`), never forked into this repo.
4. **EDGE + CLOUD** — Same code, different `just doctor` outputs. Graceful degradation when TRT unavailable.
5. **STRANDS NATIVE** — Model providers implement `strands.models.Model`; tools use `@tool` decorator.

---

## Repo Layout

```
strands-cosmos/
├── AGENTS.md                     # this file
├── README.md                     # install + quickstart + tool table
├── pyproject.toml                # v0.2.0, Apache-2.0
├── justfile                      # 50+ recipes (THE pipeline truth)
├── mkdocs.yml                    # GitHub Pages documentation site
├── strands_cosmos/               # core package
│   ├── __init__.py               # exports: 2 models + 21 tools
│   ├── cosmos_model.py           # CosmosModel (text-only Strands Model)
│   ├── cosmos_vision_model.py    # CosmosVisionModel (video+image+text)
│   ├── fix_cublas.py             # Jetson CUBLAS compatibility fix (CLI entry)
│   └── tools/                    # 21 tools (thin wrappers over justfile)
│       ├── __init__.py           # tool registry
│       ├── _common.py            # shared helpers (justfile runner)
│       ├── inference.py          # TRT server inference
│       ├── reason_hf.py          # HF Transformers direct inference
│       ├── serve.py              # TRT server lifecycle
│       ├── predict_generate.py   # Predict2.5 world model generation
│       ├── transfer_generate.py  # Transfer2.5 ControlNet video-to-video
│       ├── model_download.py     # HF model download
│       ├── quantize.py           # FP8 quantization
│       ├── export_onnx.py        # ONNX export
│       ├── build_engine.py       # TRT engine build
│       ├── post_train.py         # SFT/LoRA post-training
│       ├── distill.py            # Knowledge distillation
│       ├── curate.py             # Xenna data curation
│       ├── evaluate.py           # FID/FVD/CSE/CLIP benchmarks
│       ├── rtp.py                # GStreamer RTP frame capture
│       ├── nats_pub.py           # NATS messaging
│       ├── video_utils.py        # ffprobe + frame extraction
│       ├── image_read.py         # Base64 image read
│       ├── sysinfo.py            # GPU/platform diagnostics
│       ├── cosmos_invoke.py      # Legacy direct inference
│       └── cosmos_vision_invoke.py  # Legacy vision inference
├── examples/                     # 5 runnable examples
│   ├── 01_basic_text.py
│   ├── 02_video_caption.py
│   ├── 03_driving_analysis.py
│   ├── 04_embodied_reasoning.py
│   └── 05_tool_usage.py
├── tests/                        # pytest suite
│   ├── __init__.py
│   └── test_imports.py
├── docs/                         # MkDocs Material site
│   ├── index.md
│   ├── architecture.md
│   ├── api-reference.md
│   ├── getting-started/
│   ├── guide/
│   └── examples/
├── demo/                         # Demo GIF/video assets
├── LICENSE                       # Apache 2.0
└── sample.{mp4,png}             # Test media files
```

---

## Hardware & Platforms

| Platform | GPU | Primary Use |
|----------|-----|-------------|
| **Jetson AGX Thor** | 132GB unified | Edge deployment: TRT engines, serve, RTP capture |
| **Jetson Orin** | 32/64GB | Edge deployment (smaller models) |
| **Desktop/Cloud** | A100/H100/RTX 4090 | Training, quantization, ONNX export, HF inference |
| **Any machine** | CPU-only | Development, testing, documentation |

`just doctor` reveals what works on the current host.

---

## Dependencies Policy

**Core** (always installed): `strands-agents`, `transformers`, `accelerate`, `torch`, `torchvision`, `qwen-vl-utils`, `pillow`, `pyyaml`, `av`

**Optional extras**:
- `[vllm]` — vLLM + OpenAI client
- `[jetson]` — torchcodec companion
- `[dev]` — pytest, ruff
- `[all]` — everything

**External Cosmos repos** (cloned alongside, NOT bundled):
- `cosmos-predict2.5`, `cosmos-transfer2.5`, `cosmos-reason2`
- `cosmos-xenna` (data curation), `cosmos-rl`, `cosmos-cookbook`

Run `just setup` to auto-clone all six.

---

## Tool Architecture

Tools are **thin Python wrappers** over justfile recipes:

```
Agent calls tool → tool runs `just <recipe> <args>` → justfile executes pipeline
```

This means:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strands-labs/strands-for-cosmos](https://github.com/strands-labs/strands-for-cosmos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
