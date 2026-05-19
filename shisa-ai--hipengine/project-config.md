---
trigger: always_on
description: hipEngine is a ROCm-native inference engine built around a clean Python host and the proven gfx1100 kernel lineage from `nano-vllm-amd`. See [docs/PLAN.md](docs/PLAN.md) for architecture, phase roadmap, and LoC budgets.
---

# hipEngine - Agent Guide

hipEngine is a ROCm-native inference engine built around a clean Python host and the proven gfx1100 kernel lineage from `nano-vllm-amd`. See [docs/PLAN.md](docs/PLAN.md) for architecture, phase roadmap, and LoC budgets.

This `AGENTS.md` (`CLAUDE.md` symlinked) is read every session. It covers only ground rules that apply to every review / coding / benchmarking task. Activity-specific playbooks live in `docs/`.

Instruction precedence: if this file conflicts with platform / system / developer instructions, follow those first.

## Summary

- **Source of truth:** [docs/PLAN.md](docs/PLAN.md). Update it when architecture or phase plans move.
- **Cross-session handoff:** `WORKLOG.md`. Append-only, chronological; log decisions, commands, measurements, and next actions as they happen.
- **Testing discipline:** math changes are guilty until proven correct. Follow RED/GREEN where practical, and use `docs/TESTING.md` for fixture/oracle/gate details.
- **Evidence policy:** every performance claim carries model + quant + workload shape + hardware + exact command + result + correctness gate. No exceptions (see `docs/PLAN.md` "Evidence Policy" and `docs/BENCHMARK.md`).
- **Benchmark rollup stays current.** Every retained benchmark updates `benchmarks/README.md` (`Last updated` plus table row), `benchmarks/CHANGELOG.md` (dated one-liner with old→new metric, % delta, reason, artifact/source), and a compact artifact under `benchmarks/results/`.
- **Correctness gate for any new/ported kernel:** KL ≤ 0.05 AND top-1 agreement ≥ 90% vs `kernels/cpu_reference/` on fixture inputs.
- **Default hardware:** AMD Radeon Pro W7900, gfx1100/RDNA3. Claims about other backends require the corresponding hardware or are marked explicitly unverified.
- **Kernel R&D lives in `~/amd-gpu-tuning/`**, not here. hipEngine ingests *stable* kernels via port; micro-tuning, `rocprofv3` iteration loops, and the device-code gotcha catalog stay in the parent workspace.
- **Kernel catalog must stay current.** Before any kernel port, check `docs/KERNELS.md` and run `scripts/check_lineage.py`; update the catalog/path map if parent kernels or dispatch changed.

## Architectural Invariants

Do not drift these casually. They define what hipEngine is.

- **Torch-free runtime.** `import torch` is **not** allowed in any module reached by `hipengine.LLM.generate()`. Torch lives behind the optional `hipengine[torch]` extra and appears only as a dlpack bridge at the user boundary. Adding `import torch` anywhere on the hot path is an architectural change, not a refactor.
- **Four-axis plugin registry.** Kernels are keyed by `(backend, layer, quant, variant)`. Models, quant schemes, and layers are plugins. **Never** add `if backend == "hip_gfx1100"` or `if quant == "..."` branches in dispatch / engine / model code; register against a registry key instead. See `docs/PLAN.md` "Extensibility Design" for mechanics.
- **Fused kernels require an unfused fallback.** Every fused composite (`rmsnorm+rotate`, `gate_combine_residual`, …) must have a numerically-equivalent unfused chain registered under its primitives.
- **Kernel bodies take raw device pointers.** `__global__` signatures use `void*` / typed pointers, never `torch::Tensor`. Only the host-side launch wrappers convert.
- **`KVLiveSpans` is the attention kernel ABI, not a DMS-only concept.** Every paged-KV-write and attention-decode kernel reads `(base_offsets, live_counts, token_positions, evict_mask)`. Dense policies fill it uniformly; DMS/H2O/SnapKV fill it variably. Do not shortcut to `(block_table, context_len)`.
- **Backend tree is a peer structure.** `kernels/hip_gfx1100/`, `kernels/hip_gfx1151/`, `kernels/cuda_sm86/`, `kernels/cpu_reference/` are siblings. There is no "AMD directory".

## Key Files

| Path | Purpose |
| --- | --- |
| `docs/PLAN.md` | Architecture, phase roadmap, LoC budgets, extensibility design. |
| `docs/BENCHMARK.md` | Benchmark protocols, baselines to beat, correctness gate, artifact/rollup format. |
| `docs/TESTING.md` | RED/GREEN workflow, correctness oracles, fixture policy, validation matrix. |
| `docs/KERNELS.md` | Kernel catalog, source-lineage drift workflow, Qwen3.5/PARO optimal path map, port playbook, JIT cache gotcha, build profiles. |
| `docs/source_lineage.json` | External parent-file manifest used by `scripts/check_lineage.py`. |
| `docs/ROOFLINE.md` | RDNA3 W7900 performance model: hardware, regimes, decision tree, what-not-to-chase. |
| `AGENTS.md` / `CLAUDE.md` | Ground rules (this file). |
| `WORKLOG.md` | Append-only cross-session journal. |
| `benchmarks/README.md` | Human-readable current-fastest benchmark rollup and comparison tables. |
| `benchmarks/CHANGELOG.md` | Reverse-chronological one-line history of benchmark rollup updates. |
| `benchmarks/results/` | Compact JSON artifacts for accepted/blocked/rejected benchmark attempts. |
| `pyproject.toml` | Package metadata and extras. Do not casually add hard deps. |

## Workflow

### Before Starting

1. `git status -sb` — note unrelated changes and leave them alone.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shisa-ai/hipEngine](https://github.com/shisa-ai/hipEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
