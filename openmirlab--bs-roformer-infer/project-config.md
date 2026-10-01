---
trigger: always_on
description: bs-roformer-infer is an inference-only package wrapping BS-RoFormer
---

# bs-roformer-infer -- CLAUDE.md

## Scope

bs-roformer-infer is an inference-only package wrapping BS-RoFormer
(Band-Split RoPE Transformer) music source separation. It reprovides the
[lucidrains/BS-RoFormer](https://github.com/lucidrains/BS-RoFormer)
architecture as a pip-installable, PyTorch-based CLI + Python API with
automatic checkpoint management: no training code, no UVR GUI dependency.

Devices preserve legacy `None` auto-selection and also accept explicit `auto`,
`cpu`, `cuda`, and `cuda:N`; an explicitly requested accelerator that is
unavailable must raise, never downgrade silently. `auto` means CUDA-else-CPU.
`device="mps"` raises a clear `ValueError` -- MLX/Apple Silicon (MPS) support
was removed org-wide (2026-09-14, see brain/decisions.md); this package has no
non-CUDA accelerator path. Sessions may
release and reload models, but closed sessions are terminal. `cache_info()` and
loading share the download resolver, which reads package-owned checkpoints TOML.
Given an input folder of WAV files, it produces separated stems (vocals,
drums, bass, guitar, piano, other) plus an `*_instrumental.wav` per track.
See README.md for the public API, CLI, and full model registry.

**In scope**: inference (forward pass) only; a 34-entry registry
(`src/bs_roformer/config/checkpoints.toml`) spanning multi-stem, 53-stem mega,
four-stem, vocals, karaoke, instrumental, and de-reverb checkpoints; sha256-verified auto-download with a
configurable-dir UX contract (explicit arg > `$BS_ROFORMER_MODELS_PATH` >
`~/.cache/bs-roformer-infer`, legacy `./models` honored as a read fallback);
manual/offline install path; a download CLI (`bs-roformer-download`)
independent of the inference CLI.

**Out of scope, forever**: training/fine-tuning code, the UVR GUI itself,
hosting or mirroring checkpoint bytes in this repo's git history (weights are
always fetched at runtime -- see README's "What This Project Will NEVER
Bundle").

## Module layout

- `src/bs_roformer/bs_roformer.py`, `attend.py` -- the BS-RoFormer model
  architecture (from lucidrains/BS-RoFormer), with upstream-compatible
  `mlp_expansion_factor` pass-through for MaskEstimator checkpoint parity and
  explicit `mask_estimator_variant` selection for supported architecture heads.
- `src/bs_roformer/hyperace.py` -- the HyperACE mask-estimator variation used by
  pcunwa HyperACE v2 checkpoints. The RoFormer trunk remains in `bs_roformer.py`;
  this module owns only the segmentation head and its helper blocks.
- `src/bs_roformer/hyperace_v1.py` -- the separate HyperACE v1 segmentation head;
  v1 and v2 remain separate because their decoder state trees differ.
- `src/bs_roformer/siamese.py` -- the clean-room two-stream transformer trunk used
  by the pcunwa Siamese checkpoint.
- `src/bs_roformer/fno.py` -- the FNO mask-estimator variation used by
  pcunwa's instrumental FNO checkpoint. It reimplements the minimal FNO1d
  inference surface needed by the checkpoint instead of depending on the full
  `neuraloperator` research framework.
- `src/bs_roformer/large_inst.py` -- the Large-Inst mask-estimator variation
  used by pcunwa's `bs_large_v2_inst.ckpt`. The RoFormer trunk remains in
  `bs_roformer.py`; this module owns the four extra time/frequency Transformer
  pairs inserted before the mask MLP.
- `backbone_variant = "value_residual"` enables the experimental learned
  value-residual trunk; standard models keep their original state layout.
- `src/bs_roformer/model_registry.py` -- `BSModel` + `MODEL_REGISTRY`,
  backed by `config/checkpoints.toml` so new models don't need a code change.
  `MODEL_REGISTRY.get()` accepts slug, friendly name, or checkpoint filename.
- `src/bs_roformer/download.py` -- checkpoint/config download, sha256
  verification, models-dir resolution, and the `bs-roformer-download` CLI.
  `ensure_model_assets()` is the auto-download entry point `inference.py`
  calls on first use. Resolution precedence per asset: `config/checkpoints.toml`
  URL > packaged local file under `configs/` (configs only) >
  `DEFAULT_CKPT_BASE_URL`/`DEFAULT_CONFIG_BASE_URL` construction (legacy
  fallback path only; the old TRvlvr repo is dead -- see "Weights hosting"
  below).
- `src/bs_roformer/inference.py` -- the `bs-roformer-infer` CLI: folder-batch
  separation, chunked overlap-add, weights auto-resolve via `download.py`.
  `run_folder()` owns folder iteration, stem naming, instrumental derivation,
  the manifest, and the chunked Torch inference itself (via `utils.demix_track`)
  in one place. This used to sit behind a `backends/` package (a
  `SeparationBackend` seam + `TorchBackend` + `ChunkingPlan`) that dispatched
  between Torch and an MLX backend by name; both the MLX backend and that
  now-single-implementation seam were removed org-wide (2026-09-14, see
  brain/decisions.md) -- `run_folder()` is torch-only again, matching its
  pre-seam shape (see git history around `870c64c`) plus the manifest/
  `output_format` features added since.
- `src/bs_roformer/clean_api.py` -- `BSRoformerSession`, the public Python
  facade README leans on: explicit session lifecycle, lazy loading, and a thin
  inference call that delegates to `inference.run_folder()` (its own resident
  model, directly -- no backend indirection) and surfaces the exact output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openmirlab/bs-roformer-infer](https://github.com/openmirlab/bs-roformer-infer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
