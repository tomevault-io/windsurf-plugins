---
trigger: always_on
description: Open-source American English voice pack for Qwen3-TTS 0.6B. By Sentium.
---

# Holler

Open-source American English voice pack for Qwen3-TTS 0.6B. By Sentium.

**What this is:** A fine-tuned Qwen3-TTS-12Hz-0.6B model with 6 American English voices, optimized for local inference on Apple Silicon via mlx-audio. ~147-200ms TTFA streaming (depending on variant), ~1.7-2.4GB RAM. Published on HuggingFace as `sentiuminc/holler-0.6b` (bf16) and `sentiuminc/holler-0.6b-6bit`.

**What this is not:** A new TTS architecture. This is a fine-tune of Alibaba's [Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS) (Apache 2.0) with better English voices and a fully open training pipeline. All credit for the base model goes to the Qwen team at Alibaba. Our contribution is the voices, the Mac-focused inference setup, and the open training pipeline.

**License:** The base Qwen3-TTS model is Apache 2.0. Our fine-tune scripts and voices will also be Apache 2.0. All releases must include proper attribution to Qwen/Alibaba.

**Focus:** Mac. Training data generation runs locally on Apple Silicon via mlx-audio. Training (SFT) requires a CUDA GPU (Vast.ai). Inference target is mlx-audio on M-series Macs.

## HARD RULES

- **NEVER delete checkpoints without explicit confirmation from Chris.** Not during cleanup, not during session wrap, not ever. Checkpoints represent hours of GPU time and are irreplaceable once gone. Ask before deleting. This includes bf16 originals, quantized copies, and anything in `checkpoints/`.

**Origin:** Started as the voice component of [ivi](https://ivi.computer), a macOS notch AI assistant by Sentium. We open-sourced it because Qwen3-TTS is SOTA for local inference but ships with only 2 mediocre English voices — and nothing else fills that gap.

**Session logs:** All holler session logs go in the parent ivi repo at `ivi/logs/`, not in `holler/logs/`. Old session logs have been moved there already. `holler/logs/runs/` still holds raw training/inference output logs.

## Current State (2026-05-14)

- **Recipe:** lr=5e-7, cosine warmup (scheduler fix: `total_steps ÷ grad_accum`), 2 epochs, batch_size=2, grad_accum=4, weight_decay=0.01, **embedding normalization** (L2-norm to 10.0). All training data at uniform -22 LUFS except Nora at -24 LUFS.
- **6-voice v14 (CURRENT):** `checkpoints/holler-tts-0.6b-6bit/` and `holler-tts-0.6b-bf16/`. Kit=3000, Dakota=3001, Nora=3002, Joe=3003, Oliver=3004, Tessa=3005. Benchmark: 93% clean raw at temp 0.7, Nora 10/10 perfect. All voices within 1.2 dB LUFS of each other. Also on R2 at `r2:holler/checkpoints/holler-tts-0.6b-bf16/`.
- **v12 (previous):** 90% clean benchmark. Nora ran 3-4 dB hot. Hard clip distortion at decoder output. On R2 at `r2:holler/checkpoints/holler-6voice-v12-e1/`.
- **Training data on R2:** `r2:holler/training-data/` — all 6 voices at **-22 LUFS** (clips + refs), plus Nora at **-24 LUFS** (`audio-24lufs/` + `ref-24lufs.wav`). rclone remote `r2-sentium` configured locally.
- **R2 instance cache:** `r2:holler/instance-cache/` — pip-cache.tar.gz + models-cache.tar.gz. Setup time ~5-8 min.
- **Quantization:** 6-bit affine g64 for shipping. bf16 for best quality. **Must copy entire `speech_tokenizer/` directory** after quantization — reuse canonical copy, never re-download.
- **Inference:** Temperature 0.7 (default everywhere — code, CLI, server, generation_config.json). Codebooks 16 default (12 for fast streaming). Dynamic peak normalization at decoder (0.9 target). Streaming AGC targeting -20 LUFS in both Swift and Python. No soft clip, no per-voice gain.
- **HuggingFace:** LIVE at [`sentiuminc/holler-0.6b`](https://huggingface.co/sentiuminc/holler-0.6b) and [`sentiuminc/holler-0.6b-6bit`](https://huggingface.co/sentiuminc/holler-0.6b-6bit). Model cards with audio samples. `base_model_relation: quantized` links them.
- **Benchmark tool:** `tools/benchmark_quality.py` — Whisper medium word timestamps, gap analysis, strict WER. `--raw` and `--session` modes, `--temperature` flag, `--compare` for A/B.
- **Python venv:** `.venv` (Python 3.14, mlx-audio, mlx-whisper) and `.venv-enhance-audio` (Python 3.13, praat-parselmouth, scipy — see `requirements-enhance.txt`).

## Structure

```
holler/
├── CLAUDE.md              — This file (project instructions + state)
├── .gitignore
├── training/              — Training scripts (single-voice + multi-voice SFT)
├── inference/             — MLX + PyTorch inference, TTFA benchmarks, live demo
│   ├── server.py                   — Fast inference server (--checkpoint, --port flags)
│   ├── experiment_quant_methods.py — Quantize + benchmark all quant variants
│   ├── benchmark_katie_v6.py       — bf16 benchmark (10 texts, TTFA + peaks)
│   ├── benchmark_quant_proper.py   — all-quant benchmark with MLX Metal memory
│   ├── benchmark_sentence_queue.py — sentence-queue benchmark simulating ivi pattern
│   ├── live_demo.py                — Web UI: type text, hear it spoken
│   ├── test_mlx_inference.py       — basic MLX test
│   ├── test_mlx_streaming.py       — streaming TTFA test
│   ├── test_mlx_multivoice.py      — multi-voice test
│   ├── test_pytorch_inference.py   — GPU ground-truth test
│   └── test_epoch_sweep.py         — compare multiple epoch checkpoints
├── tools/                 — Voice design, training data generation, verification

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sentiuminc/holler](https://github.com/sentiuminc/holler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
