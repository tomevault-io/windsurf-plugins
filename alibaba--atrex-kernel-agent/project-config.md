---
trigger: always_on
description: This file defines hard behavioral constraints for the optimization workflow.
---

# GPU Kernel Optimizer — Agent Constraints

This file defines hard behavioral constraints for the optimization workflow.
The full multi-cycle workflow and terminal handoff are defined in `orchestrator/prompts/episode.md`.

## Framework Guidance

- **The V0 baseline is a pure-PyTorch reference wrapper** (correct + directly submittable), NOT yet in any optimized DSL. Migrating the body of `run()` from PyTorch to the `--framework` DSL is the *suggested* first lever of the optimization loop — do it in an early iteration, and update `solution.json` `spec.languages`/`dependencies` in the same iteration so the harness benches the real kernel.
- The `--framework` value is a **recommended optimization direction**, not a hard constraint. Sessions MAY use a different DSL or mixed approaches if evidence shows a better performance path.
- Preinstalled third-party helper libraries may be used, but the campaign environment is immutable: never
  install or locally build a package. If a library is unavailable, use existing tooling or record a blocker.
- `triton` and `gluon` belong to the same framework family (`triton/gluon`). When either is specified, both are acceptable implementation targets.
- When Triton-level optimization plateaus, the orchestrator latches a mandatory Triton→Gluon episode directive. The episode derives layouts from TTGIR, repairs the lowering through correctness and performance parity, and later episodes remain in Gluon. Do not hand-trigger conversion before the directive is active.

## Benchmark Harness Integrity

- **No hacking the evaluation script for performance.** Do NOT modify, monkey-patch, subclass, shadow, or otherwise subvert `test_kernel.py` — nor any other file/module the evaluator loads (`sol-execbench`, `torch.cuda.Event`/`time` shims, RNG/seeding utilities, the timing loop, the comparison/tolerance check) — to make a slower kernel *look* faster or to make an incorrect result *pass*. Any speedup must come from a faster `run()` on **arbitrary** inputs — not from gaming the measurement.
- **test_kernel.py is immutable for performance measurement**: DO NOT modify `test_kernel.py` to change the benchmark harness (e.g., warmup count, repetition count, `return_mode`, timing method, input shapes, or any other benchmark parameter) in order to obtain better performance numbers.
- `test_kernel.py` defines the ground-truth benchmark methodology. Any change to it invalidates cross-version comparisons.
- If a measurement methodology issue is discovered (e.g., outlier inflation, incorrect return mode), report it in `memory/v<N>.json` under `pitfalls_and_fixes` and propose the fix — but DO NOT apply the fix to `test_kernel.py` within an optimization iteration.
- **Validate + bench ONLY via `python test_kernel.py`** — it runs the real `sol-execbench` evaluator over EVERY workload in `workload.jsonl` (the full ground-truth shape set) with each workload's own tolerance. Never hand-roll a correctness test, bench a single "representative" shape, or edit the harness. A PASS here == a directly submittable solution.
- **The optimization objective is `performance.performance_score`.** Every route computes one speedup per shape and maximizes their arithmetic mean. Native Atrex-Bench uses each shape's authoritative metadata production latency as its baseline; SOL uses the evaluator's reference implementation as its baseline. Per-workload latency remains in `performance.latency_us_by_shape` for diagnosis. A version is committable only if all workloads pass and the score improves vs HEAD beyond noise.
- **The SOL ground-truth files are immutable**: never edit `definition.json`, `reference.py`, or `workload.jsonl`. Edit `kernel.py` (DPS `run()`; args = definition.inputs then definition.outputs); update `solution.json` only when languages/dependencies/entry_point change.
- **`profile_driver.py` is the immutable profiling entry point** — profilers run `python <file>`, and `kernel.py` is import-only, so profile `profile_driver.py`, never `kernel.py`. It is a protected path: choose what it drives with `PROFILE_ITERS` / `PROFILE_WARMUP` / `PROFILE_WORKLOAD_IDX` / `PROFILE_SHAPE_ID` instead of editing it, and do NOT add a `__main__` profiling block to `kernel.py` — an in-kernel entry is silently lost the next time `run()` is rewritten, leaving the profiler to capture nothing while still exiting 0. When it genuinely cannot express the case, add a fallback driver under `profiles/<dir>/harness/` and profile that file.

### Generalized Atrex-Bench problems

- When `agent_problem.json` exists, it is the authoritative public contract. Optimize across its
  complete `shape_domain`; use aggregate distribution shares only to prioritize common paths.
- Exact `shapes.json`, evaluator metadata, and per-case roofline inputs are private. Do not search
  outside the workspace for the source operator directory or reconstruct hidden cases.
- Profile a real evaluator case by selecting an opaque id from canonical
  `memory/vN.json.performance.latency_us_by_shape` with `PROFILE_SHAPE_ID`. The sandbox injects only
  that selected case into the ephemeral remote profile workspace; the driver removes its private JSON
  before importing candidate code. Profile multiple ids when distinct performance regimes matter.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alibaba/atrex-kernel-agent](https://github.com/alibaba/atrex-kernel-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
