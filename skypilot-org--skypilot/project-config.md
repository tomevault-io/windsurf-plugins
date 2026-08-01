---
trigger: always_on
description: Sky Batch distributes workloads across a pool of GPU/CPU workers via managed jobs. The coordinator runs inline on the jobs controller (no separate cluster).
---

# Sky Batch Development Context

## Architecture Overview

Sky Batch distributes workloads across a pool of GPU/CPU workers via managed jobs. The coordinator runs inline on the jobs controller (no separate cluster).

```
ds.map()
  └─ sky.jobs.launch(task with batch_coordinator metadata)
       └─ Jobs controller detects metadata flag
            └─ Runs BatchCoordinator.run() inline via asyncio.to_thread()
                 ├─ Resolve typed input readers/output writers from dicts
                 ├─ Count items & split dataset into batches
                 ├─ Persist all batches to DB as PENDING
                 ├─ Discover pool workers (ready replicas)
                 ├─ Launch long-running worker service on each worker
                 ├─ Dispatch batches via sky.exec() notify scripts
                 ├─ Track progress via batch_state DB table
                 ├─ Reduce results using output writers
                 ├─ Cleanup temp files
                 └─ Return (success) or raise (failure)
```

Client (`dataset.py`) polls `batch_total_batches` / `batch_completed_batches` from the jobs queue for tqdm progress display.

## Key Files

| File | Purpose |
|------|---------|
| `sky/batch/__init__.py` | Public API exports: `Dataset`, `load`, `save_results`, `remote_function`, reader/writer classes |
| `sky/batch/coordinator.py` | Runs inline on jobs controller; splits dataset, discovers workers, dispatches batches, reduces/cleans up results |
| `sky/batch/dataset.py` | Client-side: `Dataset.map()`, launches managed job, polls progress with tqdm |
| `sky/batch/worker.py` | Long-running HTTP service on each pool worker; hosts `/feed_batch`, `/shutdown`, `/health` endpoints |
| `sky/batch/utils.py` | Cloud storage helpers (S3/GCS), batch file management, function serialization (source-code-based) |
| `sky/batch/constants.py` | Ports, timeouts, batch naming patterns, retry settings |
| `sky/batch/io_formats.py` | Typed I/O classes: `JsonReader`, `JsonWriter`, `ImageWriter` with `InputReader`/`OutputWriter` base classes |
| `sky/batch/remote.py` | `@remote_function` decorator with closure/global validation via AST |
| `sky/jobs/controller.py` | `_run_batch_coordinator_task()` — detects `batch_coordinator` metadata, runs coordinator |
| `sky/jobs/state.py` | `batch_state` table and helper functions for batch persistence and progress aggregation |

## Worker Architecture

Workers use a **long-running service + notify pattern**:

1. Coordinator launches a persistent HTTP worker service on each cluster via `sky.exec()`.
2. For each batch, coordinator sends a lightweight notify script (also via `sky.exec()`) that `curl`s `/feed_batch`.
3. `/feed_batch` downloads the batch from cloud storage using the `InputReader`, puts it on an internal queue, and blocks until `save_results()` signals completion.
4. The notify `sky.exec()` job exits with SUCCEEDED, which the coordinator detects via `sdk.job_status()` polling.
5. On completion or cancellation, coordinator sends `/shutdown` to stop the worker service.

Worker service listens on `127.0.0.1:8290` (localhost only, not exposed to cloud).

## Typed I/O Reader/Writer System

Readers and writers are defined in `io_formats.py`:

**Base classes:**
- `InputReader(ABC)` — Abstract dataclass with `__len__()` and `download_batch(start_idx, end_idx, cache_dir)` methods
- `OutputWriter(ABC)` — Abstract dataclass with `upload_batch()`, `reduce_results()`, and `cleanup()` methods

**Built-in readers/writers:**
- `JsonReader(path)` — JSONL input; downloads full file, caches locally per job, extracts line ranges
- `JsonWriter(path, column=None)` — JSONL output; supports column filtering; reduces by merging batch files
- `ImageWriter(path, column='image')` — Writes PIL Images as individual PNGs; no reduce step needed

**Serialization:**
- `to_dict()` / `from_dict()` — Serialize to/from dicts for transport via `task._metadata` and env vars
- Custom readers/writers (defined outside the module) automatically include `_class_source` in serialization for remote reconstruction
- Workers receive formats as JSON env vars: `SKY_BATCH_INPUT_FORMAT`, `SKY_BATCH_OUTPUT_FORMATS`

**Multi-output:** `ds.map(..., output=[ImageWriter(...), JsonWriter(...)])` — each writer independently uploads, reduces, and cleans up.

**Custom readers/writers:** Subclass `InputReader`/`OutputWriter`, register via `@registry.INPUT_READER_REGISTRY.type_register(name='...')` or `@registry.OUTPUT_WRITER_REGISTRY.type_register(name='...')`. Source code is automatically serialized. See `examples/batch/custom_formats/`.

## Batch Task Detection and Metadata Flow

1. `ds.map()` creates `sky.Task(run=None)` with batch config in `task._metadata`:
   - `batch_coordinator = True` — routing flag
   - `batch_dataset_path`, `batch_output_path`, `batch_size`, `batch_pool_name`
   - `batch_serialized_fn` — base64-encoded source code
   - `batch_activate_env` — optional env activation command
   - `batch_input_format` — input reader dict
   - `batch_output_formats` — list of output writer dicts
2. In `controller.py:_run_one_task()`, the metadata flag routes to `_run_batch_coordinator_task()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skypilot-org/skypilot](https://github.com/skypilot-org/skypilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
