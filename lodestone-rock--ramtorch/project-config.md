---
trigger: always_on
description: This is the shared, repository-wide entrypoint for coding agents and human
---

# Contributing to RamTorch

This is the shared, repository-wide entrypoint for coding agents and human
contributors. Read it before editing. Keep durable guidance here and in `docs/`,
not in local chat transcripts or `agents_notepad/`. `CLAUDE.md` imports this file
for Claude Code; tools without automatic `AGENTS.md` discovery should be pointed
here explicitly. Do not duplicate these instructions in tool-specific files.

## Start here

- Read `README.md` for the project and public API overview.
- Read `docs/pipeline_parallel.md` for pipeline execution, scheduling, ownership,
  precision, profiling, and examples.
- Read `docs/offload.md` before changing weight/activation streaming or NVMe.
- Inspect `git status` and the relevant implementation/check scripts first.
  Preserve unrelated local changes; do not assume the checkout is clean.
- `agents_notepad/` and `scratchpad/` are optional, untracked local research.
  They may not exist in a fresh clone. No library code or required check should
  depend on them. Example output paths under `scratchpad/` are generated artifacts.

## Project shape

RamTorch targets a single workstation/node with ordinary PCIe GPUs and plentiful
RAM, not an NVLink cluster. Modern pipeline execution is one process with one
worker thread per stage; do not introduce process groups, NCCL, or `torchrun` as
an incidental dependency. Legacy ZeRO utilities are separate and still supported.

- `ramtorch/pipeline.py`: resident `Stage`, gradient accumulation, results/tracing.
- `ramtorch/pipeline_relay.py`: `Pipeline`, mailboxes, synchronous schedules,
  streaming inference, and training-session ownership.
- `ramtorch/pipeline_2bw.py`: persistent resident PipeDream-2BW workers, two weight
  banks, bounded admission, and optional CPU optimizer masters/state.
- `ramtorch/pipeline_2bw_trace.py`: bounded Kineto traces and worker annotations.
- `ramtorch/delayed_optim.py`: AdamEF/Lion/Muon and update-level error feedback.
  `dion3.py` adds local Dion3; `_dion3_triton.py` is its optional attributed kernel port.
- `ramtorch/offload.py`, `pipeline_offload.py`, `nvme_store.py`: streaming engines
  and backing storage. `pipeline_optimizer.py` provides parallel per-stage
  optimizer updates for synchronous pipelines.
- `ramtorch/pipeline_easy.py`: traced/automatic `PipelineModel` convenience API.
- `ramtorch/*simulator.py`: schedule/memory models, not proof of runtime overlap.
- `examples/`: runnable demonstrations, numerical oracles, and `*_check.py`
  regression scripts. Most correctness checks are standalone scripts, not pytest.
- `ramtorch/__init__.py`: public exports. Preserve legacy `ramtorch.AdamW`; it is
  distinct from `torch.optim.AdamW` and the newer `AdamEF`.

## Environment

Use an isolated environment and a PyTorch build appropriate for the machine.
The recent pipeline/2BW work was validated with PyTorch 2.8.0 / CUDA 12.8; this is
a tested configuration, not a guarantee that every older PyTorch works. Pipeline
imports require `torch.distributed.pipelining`. Do not work around an incompatible
PyTorch installation by deleting imports or weakening checks.

```bash
python -m venv .venv
source .venv/bin/activate
# Install a suitable torch/torchvision build for your CPU or CUDA first.
python -m pip install -r requirements.txt
python -m pip install -e .
# Optional, for convergence plots:
python -m pip install matplotlib
```

Run commands from the repository root with the chosen environment's `python`.
`PYTHONPATH=.` also supports running the scripts without an editable install.
Do not commit environments, datasets, traces, checkpoints, machine-specific
paths, remote-machine credentials, or external-repository symlinks.

## Non-negotiable correctness and safety

- Preserve the NVMe **training** consent gate in `offload.py`: root or sudo/wheel
  membership AND `RAMTORCH_NVME_ACKNOWLEDGE=1`; the wear warning remains unless
  the user separately sets `RAMTORCH_NVME_QUIET=1`. Never auto-set these variables,
  bypass the gate, or silently quiet the warning to make a test pass. Inference
  is deliberately ungated after its initial storage write. Do not add NVMe to
  pipeline offloading without an explicit design decision and user consent.
- CUDA enqueue completion is not device completion. Preserve producer/consumer
  event ordering, allocator/tensor lifetimes, D2H readiness, and pinned-host
  buffer ownership until asynchronous copies finish. Grad/autocast contexts
  are thread-local: configure them in workers, not just the caller.
- Do not insert device-wide synchronizations into steady-state hot paths as a
  quick race fix. Boundary drains and correctness observers are different from
  asynchronous training; document synchronization costs explicitly.
- Mean-loss gradients are scaled exactly once across equal-sized microbatches.
  Preserve `None` versus zero-gradient semantics and frozen/unused parameters.
- Inference-only execution must not allocate parameter-sized gradient state.
  Retiering is a documented between-step hard reset; preserve its optimizer
  state migration and lifetime rules.
- 2BW is a separate `Pipeline.train_session()` API, not a schedule string to
  silently substitute into `Pipeline.step()`. Maintain two compute-weight banks,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lodestone-rock/RamTorch](https://github.com/lodestone-rock/RamTorch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
