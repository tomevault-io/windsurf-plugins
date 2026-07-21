---
trigger: always_on
description: These notes capture what worked for `examples/ltx2` (LTX-2 spatial upscaler) and should be reused for future model ports.
---

# Agent Notes: PythonKit + Swift PyTorch-Parity Workflow

These notes capture what worked for `examples/ltx2` (LTX-2 spatial upscaler) and should be reused for future model ports.

## 1) Environment and launch

- Use the same run pattern from shell history:
  - `source ../ltx2/_env/bin/activate`
  - `export PYTHONPATH=/home/liu/workspace/ltx2/LTX-2/packages/ltx-core/src:${PYTHONPATH}`
  - `bazel run examples:ltx2 --compilation_mode=dbg --keep_going`
- Keep fresh repo path (`../ltx2/LTX-2`) for latest code. Do not use `../ltx2/ltx-core` for new model behavior.

## 2) PythonKit import hygiene

- PythonKit may not see all site-packages by default. Before importing model code:
  - insert `site.getusersitepackages()` into `sys.path` when missing.
  - insert `/usr/lib/python3/dist-packages` into `sys.path` when missing (needed for modules like `setuptools` transitively required by `triton`/`ltx_core` imports).
- Keep optional/heavy imports below early `exit(0)` when iterating on one submodel to avoid unrelated dependency failures.
- For CUDA-related work in this repo, prefer running direct Python probes, parity binaries, and other GPU executions **outside the sandbox**.
  - Sandboxed runs can report misleading CUDA failures such as `cuInit(0) = 100` / `cudaErrorNoDevice` even when unrestricted runs see all GPUs normally.
  - If a CUDA / PyTorch / ccv issue appears suspicious, rerun the same command unrestricted before drawing conclusions.

## 3) Swift model parity rules (must match PyTorch op order)

- Exact operator order matters more than anything else.
- For LTX-2 upscaler `ResBlock`, correct order is:
  - `conv1 -> norm1 -> SiLU -> conv2 -> norm2 -> (x + residual) -> SiLU`
- Do not switch to pre-norm pattern unless PyTorch block is pre-norm.
- Keep tensors in `N,C,D,H,W` through the graph and only permute when required by pixel shuffle layout.

## 4) Padding and layout rules

- For H/W-only padding, prefer convolution `hint` borders.
- If temporal padding is needed, use `pad`, but because pad helper is 4D-oriented:
  - reshape to 4D, pad, then reshape back.
- Validate all conv filter shapes with model semantics:
  - upsampler `Conv2d` weights from PyTorch become Swift 3D conv weights via `unsqueeze(2)` (temporal kernel size 1).

## 5) LTX-2 spatial upscaler specifics

- Checkpoint: `ltx-2-spatial-upscaler-x2-1.0.safetensors`
- Config expected in metadata:
  - `in_channels=128`, `mid_channels=1024`, `num_blocks_per_stage=4`, `dims=3`
  - `spatial_upsample=true`, `temporal_upsample=false`, `spatial_scale=2.0`, `rational_resampler=true`
- Rational x2 spatial path behavior:
  - `upsampler.conv` + pixel shuffle on H/W.
  - `blur_down(stride=1)` is effectively identity for this checkpoint.

## 6) Validation-before-export checklist

- Numeric parity checks are **mandatory** for every model conversion in this repo.
  - Do this per converted submodel first (for example: text encoder, adapter / connector, diffusion model, VAE pieces).
  - Then do an end-to-end parity check for the active exported unit when feasible.
  - Do not treat a conversion as complete just because it builds, runs, or writes a ckpt.
- Always run Swift and PyTorch on the same seeded random latent.
- Confirm output shape equality first.
- Compare sample values and record at least max-abs diff before writing ckpt.
- Prefer also recording relative diff when reference magnitudes vary significantly.
- After parity passes, ask the user before running the export step.
- When performing the final export run, rerun the numeric parity check in that same session before writing ckpt when practical.
- Only export graph (`graph.openStore(...).write(...)`) after numeric parity is confirmed and the user has approved export.

## 6.1) Model config representation

- Prefer to keep model architecture / config values on the Swift side as explicit constants.
- It is fine to group them into constant structs or tables instead of inlining them, but they should stay easy to inspect in Swift.
- Do not rely on opaque Python / JSON config reads for core architecture choices during conversion unless there is a specific reason and the user asked for it.

## 7) Iteration pattern in `examples/ltx2/main.swift`

- Add new converter block near top.
- Run only that block with `exit(0)` immediately after it.
- Keep older experiments below for reference, but not on active execution path.

## 8) LTX-2.3 environment (working setup for `examples/ltx23`)

- Repository: `/home/liu/workspace/ltx2/LTX-2`
- Create and sync dedicated Python 3.12 env:
  - `cd /home/liu/workspace/ltx2/LTX-2`
  - `uv venv -p 3.12 _env`
  - `source _env/bin/activate`
  - `uv sync --frozen --active`
- Confirmed compatible stack in this env:
  - `torch==2.9.1+cu128` (`torch.version.cuda == 12.8`)

### Build and run `ltx23` reliably

- Build first (from `/home/liu/workspace/swift-diffusion`):
  - `bazel build examples:ltx23 --compilation_mode=dbg --keep_going`
- Then run the built binary with the following environment:
  - `source /home/liu/workspace/ltx2/LTX-2/_env/bin/activate`
  - `export PYTHONPATH=/home/liu/workspace/ltx2/LTX-2/packages/ltx-core/src:${PYTHONPATH}`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liuliu/swift-diffusion](https://github.com/liuliu/swift-diffusion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
