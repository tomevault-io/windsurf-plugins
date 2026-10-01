---
trigger: always_on
description: - We own the complete serving stack, including Python orchestration, vLLM,
---

# SlimServe repository guidance

## Scope and ownership

- We own the complete serving stack, including Python orchestration, vLLM,
  ROCm/HIP code, communication paths, generated kernels, and handwritten GPU
  kernels.
- Do not stop at an upstream-library boundary or classify a kernel failure as
  external. Reproduce it, isolate the first failing operation, and fix the
  responsible layer in this repository or the owned dependency tree.
- Long-running and concurrent workloads are correctness requirements. A short
  smoke test does not replace the exact workload that exposed a failure.

## GPU implementation scope

- AITER is a supported dependency. Do not remove or replace a working AITER
  path solely to eliminate that dependency.
- AITER and its kernels are part of the stack we debug and fix when a supported
  workload exposes a failure there.
- Kernel work is in scope. Use serialized launches, device assertions,
  sanitizers, reduced reproducers, and targeted instrumentation as needed.

## Reference implementations

Use these local trees when checking algorithms, layouts, bounds, and kernel
behavior:

- `~/llama.cpp`
- `~/ds4`
- `~/QuixiCore/QuixiCore-ROCm`
- `~/QuixiCore/QuixiCore-Metal`

## SlimServe Orientation

- SlimServe is profile-driven. `slimserve/profiles.json` is the source of truth for supported models, quants, engine args, environment, and platform overrides.
- Missing model files are not a blocker. Run `slimserve <profile> ... -y`; `slimserve.fetch` downloads or resumes required files into `$SLIMSERVE_CACHE` or `~/models`.
- Do not hand-build unsupported serving commands when a profile exists. Use `slimserve <profile> --dry-run` to inspect and `slimserve <profile>` to run (serving is the default; `--serve` is still accepted).

## Agent Operating Discipline

- Read the repo before deciding. Start from `AGENTS.md`, `perf/perf.md`,
  `perf/baseline_status.md`, `perf/optimization_status.md`, the relevant
  profile in `slimserve/profiles.json`, and the local implementation paths for
  the model/platform under discussion.
- Do not confuse "the process started" with "the system works." A serving path
  is not validated until the real SlimServe profile reaches health, serves the
  benchmark workload, passes correctness checks, and produces recorded TPS.
- Do not treat vLLM interval logger output as an authoritative benchmark.
  Interval logs are diagnostics. Use exact-token harness output, raw JSON, and
  reproducible commands for baseline or comparison claims.
- Do not make a placeholder fix and describe it as the real fix. If a change is
  an interim diagnostic, say so in the notebook and keep the production target
  explicit.
- Remove or clearly quarantine failed experiments. Do not leave disabled,
  memory-heavy, or misleading alternate paths in the serving code unless they
  are intentionally retained as documented diagnostics.
- When the user points at a precedent, inspect it before implementing. For this
  repo that often means reading the optimized ROCm/Metal path, GLM 5.2 Ampere
  path, DS4 kernels, and the QuixiCore CUDA/ROCm code that is actually relevant
  to the active serving path.
- Keep model downloads out of the reasoning loop. If SlimServe owns the
  download, run SlimServe and let it download or resume; do not block kernel or
  profile work because a model file is not already present.
- State uncertainty precisely. If only startup, smoke, or synthetic parity has
  been run, call it that. Do not imply end-to-end correctness, production
  readiness, or final performance without the corresponding evidence.
- Preserve user and prior-agent changes. The worktree may be dirty; understand
  nearby edits and build on them rather than reverting unrelated work.
- Finish the loop: implement, build, smoke, run the real profile or explain the
  concrete blocker, update the performance notebook, and leave the next command
  obvious.

## Kernel Work

- Develop serving kernels in this repo first. Vendored code used by SlimServe belongs under `csrc/quixicore/` or `csrc/libtorch_stable/`; modify those copies, then port the finished used pieces to `/home/ubuntu/QuixiCore/QuixiCore-CUDA`.
- Vendor only kernels and headers on the actual serving path. QuixiCore is a large kernel library; do not copy broad directories just because they exist.
- QuixiCore, `ds4`, `llama.cpp`, and vLLM Marlin are references and inspiration, not finished answers. Study them for layouts, scheduling, quant decode, and tensor-core strategy, then implement and tune the SlimServe path that this model and hardware actually need.
- SlimServe owns the inference stack all the way down to CUDA/HIP kernels. Do not assume an upstream or vendored kernel is "already done" when profiling shows room to improve.
- The primary purpose of this repo is to optimize serving performance for the target model on the target hardware. Keep pushing until the remaining bottlenecks are measured and defensible.
- When inference is already thoroughly optimized on one platform, study that implementation before implementing another platform. Preserve the algorithmic wins, data layout choices, fusion boundaries, and profiling lessons unless the new hardware gives a measured reason to diverge.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuixiAI/SlimServe](https://github.com/QuixiAI/SlimServe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
