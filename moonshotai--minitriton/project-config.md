---
trigger: always_on
description: English | [中文](AGENTS.zh-CN.md)
---

English | [中文](AGENTS.zh-CN.md)

# MiniTriton Development Rules (AGENTS.md)

This file states the engineering principles that all development on this project (human or agent)
must follow; on conflict, this file's way of working wins. Design decisions and
measured history live in the git log and the issue tracker.

## 1. Project Structure: Simplicity and Clarity

- The package body is only `minitriton/`: `frontend/ compiler/ runtime/ device/ ops/ nn/
  autograd/ distributed/ sparse/ sched/ viz/ fusion.py compile.py precision.py`. A new module
  must first answer "why does it belong in this layer"; if you can't answer, don't add a layer
- **DSL first; primitives are the exception** (decided 2026-07-19): new features / new fused ops /
  new kernels are all written in the Python DSL and lowered through the generic pipeline; the kernel
  corpus lives in the kernel library/examples, not in the compiler. Compiler intrinsics, three-way
  split: (a) tile-vocabulary primitives (the Triton-isomorphic set: load/store/dot/mma_tile/reduce/
  scan/atomic_add etc.) — frozen; (b) expressiveness exceptions (hardware instruction-level
  choreography, operations with no tile semantics, data-structure kinds) approved case by case with
  a registered justification — if you can't answer "why can't tile express this", it's not allowed;
  **explicit scheduling constructs** (async_copy/stage_pipe/pipelined/convert_layout/fragment etc.)
  belong in area (a): generic, application-independent, not application-level intrinsics; (c)
  application-level kernels (flash/KDA kinds) are forbidden in the compiler; the migration is
  **complete**: the whole flash family (fwd+bwd × all precision tiers × plain/qkv) and all three
  KDA segments are pure DSL in `device/cuda/kernels/`;
  the row-op family ships dual forms (composed reference in `sched/rowfam.py` + fused production
  kernels, routing by measurement). General linear-algebra primitives (e.g. `solve_tril`,
  a standard building block, not application-overfit) belong to the tile vocabulary; the
  forbidden kind is application-overfit intrinsics (named after specific algorithms/models) —
  current stock: **zero**
- `benchmarks/` is folded by topic: `roofline/ matmul/ attention/ kda/ ops/ training/ probes/`;
  no new files flat at the root; one-off probes go in `probes/` with filenames starting with `_`
- `examples/` holds end-to-end demos only; `build/` is compiler intermediates (gitignored, do not
  commit)
- Large figures and data files are **locally regenerable and do not enter git** (`.gitignore`
  allowlist). Git keeps only 3 headline figures (`roofline/roofline_cuda_core.{png,svg}`,
  `roofline/roofline_tensor_core.{png,svg}`,
  `training/train_gpt_convergence_precision.{png,svg}`) plus 2 gallery figures for the
  README (`examples/gallery_rigid.{png,mp4}` via `examples/render_gallery.py`,
  `examples/train_gpt_ddp_overlay.png` via `benchmarks/training/plot_ddp_overlay.py`) —
  every tracked figure must have its regeneration command on record
- imports are always `import minitriton as mt`; intra-package relative imports preferred; no reverse
  cross-layer imports (device does not import nn, runtime does not import ops)

## 2. Testing Rules

- **Every feature must ship with a PyTorch (or numpy/fp64) diff-test**: a new op/kernel delivery =
  implementation + a reference test under `tests/` (torch is only allowed in `benchmarks/`;
  `tests/` always uses numpy/fp64 as the oracle; acceptance scripts needing a torch reference go in
  `benchmarks/`)
- State tolerances explicitly with justification (fp32 kernel vs numpy: rtol ~1e-5; tf32: 1e-2;
  bf16: 2e-2; EXC ill-conditioned regions only assert finiteness)
- The test matrix must cover: regular shapes, odd/non-divisible, non-contiguous views, broadcasting,
  boundaries (0-dim, single element), dtype variants
- Delivery bar: `pytest tests/ -q` all green + new cases pass; performance kernels must additionally
  meet "compute-bound ≥ 0.9x triton like-for-like"
- Fast parallel gate (pytest-xdist + pytest-rerunfailures are in the project env; the compile disk
  cache is concurrency-safe by design):
  `pytest tests/ -q -n 12 --deselect tests/test_streams.py --reruns 2 --reruns-delay 3`
  then `pytest tests/test_streams.py -q` serially (the streams tests are timing-sensitive under
  GPU contention). ~20 s warm vs ~75 min serial cold-after-compiler-edits
- autograd-related: `benchmarks/training/grad_check.py` (vs torch autograd, atol 1e-4) must not
  regress

## 3. Plotting Rules

- All figures go through the unified system in `benchmarks/roofline/plot_style.py`:
  - **Six-version output**: `ps.render_themes(fig_fn, out_base)` produces (dark, light) × (png,
    svg, pdf); `fig_fn` uses only `ps.*` constants and is re-callable. Light PDFs are the
    print/LaTeX-appropriate variants, dark PDFs suit slides
  - **Single color vocabulary**: `IMPL_COLORS` (minitriton hero red / torch blue / torch.compile
    green / triton purple); no new implementation colors
  - **Split legends**: `ps.split_legend` — method (color) and kernel (marker) as two independent
    legends, never mixed
  - **roofline must draw the measured roof** (self-calibrated from measurements, not spec-sheet
    numbers) **and the spec dashed line** (boost values, to preempt nitpicking); guidance

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MoonshotAI/minitriton](https://github.com/MoonshotAI/minitriton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
