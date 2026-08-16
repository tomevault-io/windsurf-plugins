---
trigger: always_on
description: Use this guide when diagnosing or tuning a NeMo Curator pipeline running on the
---

# Ray Data Backend — Agent Tuning Guide

Use this guide when diagnosing or tuning a NeMo Curator pipeline running on the
Ray Data backend. Start from measured scheduler behavior; do not apply a setting
only because it helped another pipeline.

**Supported baseline:** Curator requires Ray 2.57.0 or later. The diagnostic shim
in `diagnostics.py` targets exactly Ray 2.57.0 unless the installed Ray version
already provides the diagnostics natively. Ray Data defaults and private APIs can
change between releases, so re-check source before carrying this guidance forward.

## Start here

1. Treat a Curator `Task`, not a document or ordinary Ray row, as the unit of work.
2. Enable diagnostics and inspect `ray-data.log` before changing worker counts,
   concurrency, memory, or backpressure.
3. Distinguish starvation, actor saturation, resource admission, and downstream
   backpressure. Similar GPU utilization can arise from different causes.
4. Change one lever per run and compare scheduler events, operator timing, task
   timing, object-store usage, and GPU telemetry—not wall time alone.

## Curator execution model

Ray Data parallelizes map work over blocks, then forms row batches within those
blocks. Curator constructs its initial dataset with:

```python
ray.data.from_items(tasks, override_num_blocks=len(tasks))
```

This creates one opaque Task row per block. Fanout stages repartition their outputs
back to one row per block. In practice, **one row = one block = one Task**.

The default stage `batch_size=1` passes one Task to each stage call. Ray cannot see
or split records contained inside that Task. If a task is too large, too small, or
too slow, tune Curator task partitioning and per-task payload size rather than Ray
block-size knobs. Pipeline completion calls `take_all()`, so final Task payloads are
collected back to the driver.

### Task stages and actor stages

`RayDataStageAdapter` applies every stage with `map_batches` and chooses the compute
strategy as follows:

| Stage | Curator selection | Worker behavior |
|---|---|---|
| Actor | Overrides `setup()`, or requests both CPU and GPU; can be forced with `IS_ACTOR_STAGE=True` | Persistent state and `ActorPoolStrategy`; fixed or autoscaling pool |
| Task | Stateless and not selected as an actor; can be forced with `IS_ACTOR_STAGE=False` | `TaskPoolStrategy`; no actor startup or actor autoscaler |

For a task stage, `num_workers=N` caps the task pool at N; without it, Ray Data
controls parallelism through resources and backpressure. For an actor stage,
`num_workers=N` creates a fixed pool. Otherwise Curator passes `MIN_WORKERS`,
`MAX_WORKERS`, and `INITIAL_WORKERS` to an autoscaling actor pool.

## Log-first tuning workflow

### 1. Record the pipeline shape

Before tuning, record:

- task count, representative serialized task size, and work contained in each Task;
- stage `batch_size`, task-versus-actor selection, CPU/GPU request, and worker bounds;
- `max_concurrency` and the global `max_tasks_in_flight_per_actor` override;
- cluster CPU/GPU resources, object-store size, and Ray temporary directory;
- per-stage processing time, gaps between tasks, and GPU utilization/power.

Without this baseline, a faster run can be a workload, cache, GPU-power, or startup
artifact rather than a scheduler improvement.

### 2. Enable and find diagnostics

Set the opt-in environment variable before starting the driver:

```bash
export NEMO_CURATOR_RAY_DATA_DIAGNOSTICS=1
```

The values `true`, `yes`, and `on` also enable it.

The shim emits structured logfmt events through the `ray.data` logger to:

```text
$RAY_TEMP_DIR/session_latest/logs/ray-data/ray-data.log
```

Set `RAY_TEMP_DIR` with `RayClient(ray_temp_dir=...)` or
`SlurmRayClient(ray_temp_dir=...)`; the default is `~/.ray`. Ray can also resolve
the directory with `ray.data._internal.logging.get_log_directory()`.

Events are emitted on scheduler **state changes**, not on every tick. Actor counts
and utilization in an event are snapshots, not a time series. Pair them with Ray
operator timing and GPU telemetry.

### 3. Classify the bottleneck

| Symptom | Evidence to inspect | Likely interpretation | First controlled experiment |
|---|---|---|---|
| GPU actors are idle and `queued_input_blocks=0` | Upstream operator timing and admission events | GPU stage is starved | Adjust task partitioning or upstream parallelism; verify the producer is not blocked |
| Inputs are queued but work is not admitted | `scheduling_reason`, remaining budget, actor slots | Resource, capacity, concurrency, or actor-slot limit | Change the indicated limit only |
| Autoscaling pool stays near its minimum despite backlog | `utilization`, `tasks_in_flight`, scheduling reason | In-flight/concurrency ratio cannot reach scale-up threshold, or backpressure suppresses scaling | Fix the ratio or the blocking condition |
| Producer stops while object-store bytes rise | Resource admission reason and internal/output bytes | Pending output or total object-store budget is exhausted | Reduce per-task payload or increase object-store capacity |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-NeMo/Curator](https://github.com/NVIDIA-NeMo/Curator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
