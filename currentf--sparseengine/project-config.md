---
trigger: always_on
description: If SparseEngine has helped you, please give it a [Star on GitHub](https://github.com/CURRENTF/SparseEngine); it means a lot to us.
---

# Repo Skills

If SparseEngine has helped you, please give it a [Star on GitHub](https://github.com/CURRENTF/SparseEngine); it means a lot to us.

This repository includes repo-local Codex skills.

## Available skills

- `add-sparse-method`: Add or refactor a first-class SparseEngine sparse method following this repo's architecture. Use when Codex needs to introduce a new `sparse_method`, move method logic out of `attention.py` or `utils/`, add method-specific cache metadata or decode-time view building, and preserve the cache-manager-first design. File: `.agents/skills/add-sparse-method/SKILL.md`
- `code-review`: Review SparseEngine diffs for correctness, sparse-runtime and operator architecture, scheduling semantics, reproducibility, performance, and tests. Use when reviewing PRs, git diffs, sparse method integrations, operator/provider or kernel changes, cache-manager or scheduler changes, benchmark/evaluation scripts, OpenAI serving changes, or when the user asks for a code review. File: `.agents/skills/code-review/SKILL.md`
- `review-operator-organization`: Review operator/provider boundaries, device capability selection, kernel ownership, dependency compatibility, weight layouts, fallback semantics, and validation. Use for changes under `operators/`, `platforms/`, Triton kernels, external kernel integrations, or model-to-operator call sites. File: `.agents/skills/review-operator-organization/SKILL.md`
- `optimize-sparseengine-kernel`: Find, implement, tune, profile, and integrate SparseEngine GPU kernels across Triton, TileLang, CUDA/CuTe, and external SGL providers. Use for kernel hotspots, fusion, correctness baselines, microbenchmarks, Nsight Compute analysis, provider integration, or matched end-to-end performance validation. File: `.agents/skills/optimize-sparseengine-kernel/SKILL.md`

## How to use

- In this repo, invoke the sparse-method skill as `$add-sparse-method`.
- In this repo, invoke the review skill as `$code-review`.
- Invoke focused operator reviews as `$review-operator-organization`;
  `$code-review` loads it automatically for relevant diffs.
- Invoke the end-to-end kernel workflow as `$optimize-sparseengine-kernel`; it
  loads only the selected DSL and profiling references.
- Keep method-specific runtime state in `src/sparseengine/engine/cache_manager/`.
- Keep `src/sparseengine/layers/attention.py` generic and hook new methods through shared cache-manager interfaces when possible.

# Task Running Rules

1. Before running a task, check whether each device is idle. Select an idle device when one is available. If all devices are busy, wait first; if the wait becomes too long, report the situation instead of starting the task on a busy device. Ignore the above requirements when the user indicates that the GPU can be shared with other processes.
2. Do not hardcode private paths (including local machine paths and remote paths) in test scripts; pass them via variables or arguments instead. Scripts located under `scripts/tmp/` are exempt from this restriction.
3. When using a conda environment, activate it or use `conda run`; invoking only its absolute `python` path does not expose environment-provided executables such as `ninja` to child processes.

# Standardized Efficiency & Performance Benchmark Suite

The canonical runbooks are:

- [English efficiency benchmark runbook](docs/en/benchmarking/efficiency.md)
- [简体中文效率基准运行手册](docs/zh/benchmarking/efficiency.md)

Follow the runbook's matched-trace, idle-GPU, artifact-validation, and metric-
interpretation rules. Do not treat sampled GPU activity as theoretical MFU/MBU;
use the documented Nsight diagnostic for kernel-timeline attribution.

## Benchmark Entrypoints and Shared Statistics

- For paper comparisons, use the [efficiency runbook](docs/en/benchmarking/efficiency.md)
  for metric definitions and check out `dev-paper-branch` for experiment-specific
  configurations and workflows. Main decode results use continuous windows
  with boundary-only synchronization,
  preserving supported async/overlap execution. Step-synchronized runs are diagnostics.
- Use `scripts/benchmarks/run_efficiency_probe.sh` (idle-GPU checks and sweeps)
  or `benchmark/efficiency/bench_probe.py` (explicit engine/TP configuration)
  for request TTFT/TPOT and end-to-end throughput. Do not create another runner
  for a new model, method, or shape; extend the existing arguments if needed.
- All latency distributions and stage-throughput math belong in
  `benchmark/efficiency/metrics.py`. Existing runners import it. Reaggregate
  probe artifacts with
  `python3 benchmark/efficiency/metrics.py <RUN_DIR>/request_samples.jsonl`.
  This command prints JSON and does not require CUDA or overwrite old artifacts.
- Use `benchmark/microbench.py` for separately timed prefill/decode engine
  steps. CUDA step synchronization is opt-in via `--synchronize_step_timing`,
  for stage diagnostics only; without it, synchronized stage rates are null.
  Do not add per-step CUDA synchronization to request TTFT/TPOT measurements:
  timestamp token publication events and preserve the engine's execution rhythm.
  The old
  `scripts/benchmarks/bench_sparse_engine.py` command remains a compatibility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CURRENTF/SparseEngine](https://github.com/CURRENTF/SparseEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
