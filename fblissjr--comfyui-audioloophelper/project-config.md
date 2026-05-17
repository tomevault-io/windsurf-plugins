---
trigger: always_on
description: Last updated: 2026-05-16
---

# ComfyUI-AudioLoopHelper

Last updated: 2026-05-16

ComfyUI nodes that automate loop timing + audio analysis for full-length music video generation with LTX 2.3. Core pattern: `AudioLoopController` drives stride from integer latent counts, audio is frozen via `noise_mask=0`, prompts pre-encoded once outside the loop (CLIP must never enter the loop body). **Start here:** `docs/architecture_overview.md`; task-first nav at `docs/README.md`.

## Contents

1. [Commands](#commands) — pytest, audit, common apply-script invocation
2. [Architecture](#architecture) — files, nodes, entry points
3. [Critical constraints](#critical-constraints) — split by topic
4. [ComfyUI gotchas](#comfyui-gotchas)
5. [Init image conditioning + IC-LoRA paths](#init-image-conditioning--ic-lora-paths)
6. [Working with Claude across sessions](#working-with-claude-across-sessions)
7. [Documentation conventions](#documentation-conventions)
8. [Pending review](#pending-review) — capture-then-review staging

## Commands

```bash
# Full test suite
uv run --group dev --group analysis python -m pytest tests/ -v --rootdir=.

# Workflow audit (sweeps example_workflows/ + audited subset of experimental/)
uv run --group dev python scripts/audit_workflows.py

# Apply script — typical shape (every apply_*.py supports these)
uv run --group dev python scripts/apply_<X>.py            # apply
uv run --group dev python scripts/apply_<X>.py --dry-run  # show changes
uv run --group dev python scripts/apply_<X>.py --revert   # undo
```

Add `--group experiments` for autoresearch contract tests (`tests/test_autoresearch.py`); they skip on clones without duckdb. Subtree CLAUDE.md files cover deeper conventions: working in `scripts/`, `tests/`, or `internal/autoresearch/` loads the matching subtree CLAUDE.md automatically.

## Architecture

Runtime files: `nodes.py` (core loop), `nodes_analysis.py` (torchaudio audio analysis), `nodes_sage.py` (sage attention), `nodes_validation.py` (config validator). Entry point: `comfy_entrypoint()` in `nodes.py`.

Core nodes (per-node role + wiring in each class's docstring; full reference at `docs/reference/ltx23_model_reference.md`):

- **Loop spine**: `AudioLoopController`, `LoopIterationStamp`, `IterationCleanup`, `AudioLoopPlanner`, `AudioDuration`
- **Prompt schedule**: `TimestampPromptScheduleBatchEncode` + `ConditioningSelectByIteration` (current) / `TimestampPromptSchedule` + `CachedTextEncode` (legacy; don't wire in loop body)
- **Keyframe schedule**: `KeyframeLatentScheduleBatchEncode` + `LatentSelectByIteration` (current — VAE-encodes once outside loop) / `KeyframeImageSchedule` + `ImageBlend` (legacy; per-iter VAE)
- **Latent ops**: `LatentContextExtract`, `LatentOverlapTrim`, `LatentTemporalMask` (retake; `edge_taper_seconds` for soft boundary), `LatentSeamZoneMask` (multi-band mask centered on iteration boundaries — pairs with `scripts/diagnose_overlap_seams.py`), `LatentFrameCount` (sizes empty audio latent for upscale + seam), `TrimImageBatchToAudio` (F14), `TrimVideoLatentToAudio` (A/B staged), `LTXHeadTrim` (image+audio composite — drops first N latent-frames' pixel + matching audio span; default 0 = no-op), `RunIdPrefix` (F15)
- **Attention + profiling + blend**: `AudioLoopHelperSageAttention` (default `auto` as of 2026-05-15 via `scripts/apply_sage_mode_auto.py`; `auto_mask_aware` was prior default — runtime-equivalent on audio-loop workflows since the masked path doesn't fire there, and `auto` is what benchmark workflows need), `ProfileBegin`/`IterStep`/`End`, `ConditioningBlend` (works with Gemma 3 + CLIP)
- **Step-skipping cache**: `LTXVideoEasyCache` (experimental, default off)
- **Dimension SSoT**: `LTXFramePlanner` — see `docs/reference/frame_planner_reference.md`

Analysis (`nodes_analysis.py`, torchaudio only): `AudioPitchDetect` → F0 + vocal-fraction; pairs with `ConditioningBlend.blend_factor`.

## Key patterns

- `AUDIO = {"waveform": Tensor, "sample_rate": int}`. Duration = `waveform.shape[-1] / sample_rate`.
- **Stride from integer-latent counts**, not widget seconds. `overlap_seconds` widget is a target; node emits the effective quantized overlap. Prevents lip-sync drift across overlap-widget changes. Math + wiring + audits: `docs/reference/audio_loop_controller.md`.
- LTX 2.3 text encoder is Gemma 3, NOT CLIP. Format: `[tensor, {"attention_mask": mask}]`, no pooled.
- Video VAE formula: `latent = (pixel - 1) // 8 + 1`. Not `pixel // 8`.
- `noise_mask=0` = fixed context; `mask=1` = regenerate. Audio frames always 0, video 1, overlap context 0. Setters/strippers + decision table: `docs/reference/noise_mask_semantics.md`.
- Guide chaining: multiple `LTXVAddLatentGuide` / `LTXVAddGuideMulti` (up to 20) accumulate via `keyframe_idxs`; `LTXVCropGuides` strips them.
- **CFG-analog amplification**: feed `(positive_with_X, positive_without_X)` to `CFGGuider` to amplify any conditional via existing CFG math. POC: `scripts/apply_ttc_iclora_amplification_poc.py`. Mechanism + decision table + failure modes: `docs/reference/cfg_analog_amplification.md`.

## Critical constraints

### Audio + latent topology

- **Audio path is sacred.** `LTXVAudioVAEEncode → LTXVConcatAVLatent`; never feed visualizations into the video latent (heatmap frames result).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fblissjr/ComfyUI-AudioLoopHelper](https://github.com/fblissjr/ComfyUI-AudioLoopHelper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
