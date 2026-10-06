---
trigger: always_on
description: This supplements the root `.claude/CLAUDE.md` which covers core torchtitan
---

# GraphTrainer Development Guide

This supplements the root `.claude/CLAUDE.md` which covers core torchtitan
conventions (code style, naming, testing, PR expectations, etc.). Rules there
apply here too unless overridden below.

## Graph Pass Signature

All graph passes must follow this signature:
```python
def my_pass(gm: torch.fx.GraphModule, example_inputs, *, other_kwargs) -> torch.fx.GraphModule:
```
- The first two positional args are always `(gm, example_inputs)`.
- Any additional parameters must be **keyword-only** (after `*`).
- The pass must return the (possibly transformed) `GraphModule`.
- Passes that don't need `example_inputs` should still accept it (use `example_inputs=None`).

## Pass Configuration

Per-pass configuration (e.g. `static_input_indices` for cudagraph) must be
bound during pass construction in `construct_default_graph_passes` via
`functools.partial`, **not** threaded through `apply_graph_passes` as
parameters. The apply function is a generic pass runner and must not contain
pass-specific arguments.

## Pass Tiers

Graph passes are structured into two tiers:

1. **Default passes** (`passes.py`, `remove_noop_passes.py`, etc.) — always
   applied. These are numerics-preserving: cleanup, memory policy, bucketing,
   async TP, FlexAttention regional Inductor (required for bitwise match with
   eager).

2. **Performance passes** (`performance_passes.py`) — opt-in via
   `--compile.numerics_changing_optim`. These improve performance but may change numerics
   compared to the uncompiled path (e.g. RMSNorm Inductor fusion).

When adding a new pass, put it in `performance_passes.py` if it changes
numerics; otherwise put it in `passes.py` or a dedicated file like
`remove_noop_passes.py`.

## EP Overlap Trace Contract

EP overlap graph chunking is intentionally coupled to tracing through the
`ep_overlap` trace-input preparer. The preparer marks token-grid dimensions
before `minimal_fx_tracer` fakeifies inputs; the chunk pass later uses those
symbols as its source of truth. When changing EP-overlap input preparation,
dynamic-shape handling, or graph chunking semantics, update the README contract
and the trace/chunking tests together.

## Memory Policy Framework

PyTorch's module-level `torch.utils.checkpoint` and eager SAC make
coarse save-or-recompute decisions for an entire module's output, and
composing activation checkpointing with CPU offload is difficult. This
framework instead operates on the FX graph at individual tensor
granularity: each activation can independently be saved, recomputed, or
offloaded, and different strategies mix freely within a single layer.

`tag_with_memory_policy_pass` is the unified entry point. It is a
two-step process:

1. **Tag nodes.** Each saved forward activation is tagged with one of:
   - `MUST_SAVE` — keep the activation in GPU memory.
   - `MUST_RECOMPUTE` — discard and recompute during backward.
   - `MUST_CPU_OFFLOAD` — offload to CPU, reload before backward.

   Tagging can be done manually (per-node annotations) or with an
   advanced solver algorithm that optimizes the save/recompute/offload
   split based on memory budget and compute cost.

2. **Act on tags.** Two passes run unconditionally after tagging (both
   are no-ops when no nodes carry the relevant tag):
   - `apply_cpu_offload_pass` — inserts offload/reload/wait ops for
     `MUST_CPU_OFFLOAD` nodes.
   - `selective_activation_remat_pass` — duplicates `MUST_RECOMPUTE`
     ops in front of their backward consumers and erases originals whose
     consumers were all backward.

The `--compile.memory_policy` config selects the tagging strategy.
New policies (e.g. budget-aware mixed SAC + offload) should be added
as new branches in `tag_with_memory_policy_pass`.

**NUMA binding for CPU offload:** On multi-NUMA machines (e.g. GB200
NVLink-C2C), D2H/H2D bandwidth is ~350 GB/s NUMA-local vs ~120 GB/s
cross-NUMA. `Trainer` automatically applies NUMA binding
(`AffinityMode.NODE`) on CUDA hardware at init, pinning each worker
to the NUMA node of its GPU. Falls back gracefully on non-CUDA
hardware or when `numactl` is unavailable.

**Inspecting tags:** `log_activation_memory_policy` (`log_activation_memory_policy.py`)
prints all forward nodes consumed by backward, grouped by layer with
identical patterns consolidated. Shows memory, dtype, policy
(SAVE/RECOMPUTE/OFFLOAD), shape, submodule, target op, and source location.
It runs automatically at the end of `tag_with_memory_policy_pass`,
logging to both `logger.debug` and tlparse (via `trace_structured`).

## Don't Modify Core for This Experiment

Do not add `if graph_trainer:` branches to `torchtitan/train.py`
or other core files. GraphTrainer extends `Trainer` and overrides behavior through
subclassing.


### Local development (debug models, 8 GPUs)

**Run all commands from the repo root.** Use the root `./run_train.sh` with
`MODULE=graph_trainer.llama3` (or `.deepseek_v3`, `.qwen3`). The 8B/16B configs
use `hf_assets_path` relative to the repo root.

For CooR precompile workflows that need `--virtual-local-rank`, use
`torchtitan/experiments/graph_trainer/run_train_precompile.sh` instead.

```bash
# Llama3 with FSDP + TP
NGPU=8 MODULE=graph_trainer.llama3 CONFIG=graph_trainer_llama3_debugmodel \
    ./run_train.sh \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [one2piece2hello/faibench_Frontier_InfraBench](https://github.com/one2piece2hello/faibench_Frontier_InfraBench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
