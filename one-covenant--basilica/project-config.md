---
trigger: always_on
description: This repository is the customer-facing surface for Basilica: the `basilica` CLI, the Python and Rust SDKs, and runnable examples. All agent work should target this surface.
---

# AGENTS.md

This repository is the customer-facing surface for Basilica: the `basilica` CLI, the Python and Rust SDKs, and runnable examples. All agent work should target this surface.

Start here:

- CLI source: `crates/basilica-cli/src/`
- Python SDK: `crates/basilica-sdk-python/`
- user docs: `docs/GETTING-STARTED.md`, `docs/quickstart.md`, `examples/`

## What To Use

Use these repo-local skills for agent work:

- `basilica/.claude/skills/basilica-account-ops/SKILL.md`
  - login, device-code auth, API tokens, balance, TAO funding, deposit tracking
- `basilica/.claude/skills/basilica-rentals-ops/SKILL.md`
  - machine discovery, SSH keys, direct rentals, volumes, copy/exec/ssh, teardown
- `basilica/.claude/skills/basilica-serverless-ops/SKILL.md`
  - `basilica deploy`, `summon`, vLLM, SGLang, OpenClaw, Tau, scale/logs/share-token
- `basilica/.claude/skills/basilica-sdk-ops/SKILL.md`
  - Python automation, SDK caveats, blocking deploy semantics, low-level API gaps

For a single end-to-end playbook with copyable flows, use:

- `basilica/docs/agent-cloud-ops.md`
  - auth, funding, rentals, deploys, inference, OpenClaw, Tau, SDK, cleanup

## Routing

If the request is about:

- account setup, credits, deposits, or top-up: use `basilica-account-ops`
- direct GPU/CPU machines, SSH access, training boxes, or persistent rented hosts: use `basilica-rentals-ops`
- HTTP services, inference endpoints, public URLs, or self-hosted apps: use `basilica-serverless-ops`
- Python code, notebooks, scripts, CI automation, or typed programmatic access: use `basilica-sdk-ops`
- distributed PyTorch / NCCL collective workloads (DDP, DiLoCo, FSDP), multi-rank training with `torch.distributed`: use the `@basilica.distributed` SDK surface (see "Distributed Training" below)

## Distributed Training

When the user wants multi-rank PyTorch training (DDP, DiLoCo, FSDP, any
NCCL-collective workload), the canonical SDK surface is the
`@basilica.distributed` decorator. ONE entry point handles both shapes:

```python
import basilica
from basilica import ProviderFilter, WorldSize


@basilica.distributed(
    name="dlc-...",
    image="ghcr.io/one-covenant/basilica/basilica-distributed-trainer:latest",
    world_size=WorldSize(min=2, target=4, max=4),
    gpu_count=1,
    gpu_models=["A100"],
    provider_filter=ProviderFilter(include=["hyperstack", "verda"]),
    topology_spread="pack",        # required for direct WG mesh on NCCL
    bench=True,                    # opt-in NCCL bandwidth probe
)
def train():
    import os, torch, torch.distributed as dist
    dist.init_process_group(backend="nccl")
    # ... uses os.environ['RANK'] / ['WORLD_SIZE'] / ['LOCAL_RANK'] ...
    dist.destroy_process_group()


with train() as training:               # auto-cleanup on scope exit
    training.wait_until_complete(timeout=1800)
    print(training.bench)               # BenchResult | None
```

For BYO launchers (torchrun, mpirun, accelerate), pass `command=[...]`
and `basilica.distributed(...)` returns a `DistributedTraining` directly
(factory mode):

```python
training = basilica.distributed(
    name="dlc-byo-launcher",
    image="ghcr.io/one-covenant/basilica/basilica-distributed-trainer:latest",
    command=[
        "torchrun",
        "--rdzv-backend=etcd",
        "--rdzv-endpoint=$BASILICA_RDZV_ENDPOINT",
        "--rdzv-id=$BASILICA_RDZV_ID",
        "--nnodes=$BASILICA_WORLD_TARGET",
        "--nproc-per-node=$BASILICA_GPUS_PER_POD",
        "/workspace/my_training.py",
    ],
    world_size=WorldSize(min=2, target=2, max=4),
    gpu_count=1,
    gpu_models=["A100"],
    provider_filter=ProviderFilter(include=["hyperstack", "verda"]),
    topology_spread="pack",
)
with training:
    training.scale(target=3)
```

Rules:

- ONE canonical surface: `@basilica.distributed` (decorator) or
  `basilica.distributed(command=[...])` (factory). Both return a
  `DistributedTraining` context manager.
- Use `with training:` for mid-run orchestration (`scale()`,
  `wait_until_*`, `logs()`, `bench`). Bare call is fire-and-forget with
  auto-cleanup.
- `bench=True` (bool) opts in to the per-UD NCCL bandwidth probe. Read
  via `training.bench` (`BenchResult | None`); `None` means "no
  measurement" regardless of why. Use `training.bench_diagnostics` only
  for the rare debug case.
- DO NOT use `client.deploy_distributed_managed(...)`, `bench="on-start"`,
  `training.bench_status`, or `training.wait_until_bench_complete()` --
  all four were REMOVED in 0.30.0 (SDK-S7) and now raise
  `AttributeError` / `ImportError` / `ValidationError`. The
  `@basilica.distributed` decorator is the ONE canonical surface; see
  the SDK README's "Migration from the legacy surface" table for the
  per-symbol mapping.
- Worked examples: `examples/20_distributed_diloco.py` (decorator +
  bench + DiLoCo), `examples/21_distributed_torchrun.py` (BYO command +
  mid-run scale), `examples/22_distributed_with_bench.py` (bench-result
  inspection + JSON dump).
- User-facing runbook: `docs/runbooks/USER-RUNBOOK-DISTRIBUTED-NCCL.md`
  on basilica-backend. Custom-image guide:
  `docs/runbooks/BRINGING-YOUR-OWN-TRAINER-IMAGE.md`.

## Canonical Agent Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [one-covenant/basilica](https://github.com/one-covenant/basilica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
