---
trigger: always_on
description: GEMINI captures training state into host memory, replicates it to a peer, and optionally persists it to node-local storage.
---

# GEMINI

GEMINI captures training state into host memory, replicates it to a peer, and optionally persists it to node-local storage.
It provides a fast recovery tier for PyTorch, TorchTitan, Megatron Core, and DeepSpeed while leaving durable global checkpointing to the training framework.

For the system design and evaluation, see [GEMINI: Fast Failure Recovery in Distributed Training with In-Memory Checkpoints](https://doi.org/10.1145/3600006.3613145).
This guide defines the operational contract of the current implementation.

## Architecture

Checkpoint work is pipelined after each capture:

1. GEMINI copies the current model, optimizer, and caller-owned state from GPU to pinned CPU memory.
2. A background completion worker waits for the host copy and immediately starts replication to a paired rank.
3. Node-local serialization runs independently at the configured flush cadence.

The asynchronous GPU-to-CPU copy records a CUDA event.
`maybe_wait()` waits for the completion worker and therefore guarantees that peer replication has been launched.
This boundary prevents a checkpoint from mixing state from two optimizer steps.

Replication can span several training steps, but it must complete before the source buffer is reused at the next capture.
Transfers are divided into fixed-size chunks to bound head-of-line interference with training communication.

## Buffer Layout

Replicated mode allocates four host slots lazily on the first capture:

| Slot | Role |
|---|---|
| `own_current` | Receives the current GPU-to-CPU copy and becomes the replication source |
| `own_previous` | Retains the prior completed local recovery copy |
| `peer_current` | Receives the peer's current replica |
| `peer_previous` | Retains the prior completed peer recovery copy |

Each slot owns its tensor buffers, structural metadata, step, and non-tensor state.
Non-tensor state includes scheduler, sampler, RNG, and training-position data supplied by the framework adapter or `extra_state_fn`.
The metadata travels with peer replication so recovery reconstructs the complete captured state.

HSDP can skip explicit peer replication because the replica dimension already holds corresponding shards.
In that mode GEMINI uses two own slots.

## Peer Replication

`replication_jump` pairs ranks separated by a fixed world-rank distance.
The default selects one visible node width so paired ranks normally reside on different hosts.

For a world size of 16 and `replication_jump=8`, rank `0` pairs with rank `8`, rank `1` with rank `9`, and so on.
Validate this assumption when rank placement is not contiguous by node.

The built-in replication path uses a dedicated Gloo process group.
It is validated for correctness over TCP and is not a line-rate RDMA implementation.
Production deployments can use manager-driven Torch Distributed or NIXL transfer APIs for replacement workflows, but automatic checkpoint replication currently uses Gloo.
Manager-driven transfers bind a key to endpoint and tensor metadata, verify per-chunk checksums, and use bounded waits.
The torch-distributed backend communicates on a dedicated Gloo group and requires both endpoints; fallback from one-sided NIXL is therefore explicit rather than automatic.

`replication_chunk_size` limits the largest in-flight transfer unit.
Choose it from measured training communication slack rather than assuming a fixed network rate:

```python
from lm_resiliency import estimate_chunk_size

chunk_size = estimate_chunk_size(
    nic_bandwidth_gbps=400,
    layer_compute_ms=9.4,
    ag_time_ms=1.0,
    max_ag_delay_fraction=0.05,
)
```

The default is 16 MiB.
The estimator returns a bound derived from the supplied bandwidth and prefetch timing; it is not an automatic network profiler.

## Configuration

```python
from lm_resiliency import InMemoryCkptConfig

config = InMemoryCkptConfig(
    enable=True,
    interval=10,
    replication_jump=-1,
    replication_chunk_size=16 * 1024 * 1024,
    disk_flush_interval=100,
    disk_folder="./checkpoints",
    run_id="training-run-2026-08-15",
    verify_integrity=False,
    skip_replication_if_hsdp=True,
    pin_memory=True,
)
```

| Field | Meaning |
|---|---|
| `interval` | Capture cadence in optimizer steps when GEMINI runs independently |
| `replication_jump` | Peer rank spacing; `-1` uses the visible GPU count |
| `replication_chunk_size` | Bytes per replication send |
| `disk_flush_interval` | Node-local flush cadence; `0` disables periodic flush |
| `disk_folder` | Node-local checkpoint directory |
| `run_id` | Stable identity required to resume this run's node-local files |
| `verify_integrity` | Store and verify CRC-32 for serialized shards |
| `skip_replication_if_hsdp` | Use natural HSDP replicas instead of explicit peer transfer |
| `pin_memory` | Allocate page-locked host buffers for asynchronous copies |

The unified `enable_resiliency(..., interval=N)` call overrides the component interval.

Set `run_id` to the same non-empty value on every rank and reuse it only for an
intentional resume. When it is omitted, GEMINI uses `LM_RESILIENCY_RUN_ID` or
torchrun's `TORCHELASTIC_RUN_ID`; without either launcher identity it coordinates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LMResiliency/lm-resiliency](https://github.com/LMResiliency/lm-resiliency) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
