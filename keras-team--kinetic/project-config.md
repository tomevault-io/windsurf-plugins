---
trigger: always_on
description: Kinetic lets users execute Keras/JAX workloads on cloud TPUs and GPUs via a single decorator (`@kinetic.run()`). It handles infrastructure provisioning, container building, job submission, and result retrieval on GCP.
---

# AGENTS.md — Kinetic

## Project Overview

Kinetic lets users execute Keras/JAX workloads on cloud TPUs and GPUs via a single decorator (`@kinetic.run()`). It handles infrastructure provisioning, container building, job submission, and result retrieval on GCP.

## Architecture

```
kinetic/
├── core/           # @run decorator, accelerator registry & parser
├── backend/        # Job execution backends (GKE, Pathways)
├── data/           # Data class for declaring data dependencies
├── infra/          # Docker container building & caching
├── runner/         # Remote worker entrypoint (runs inside container)
├── utils/          # Serialization (packager) and Cloud Storage helpers
├── cli/            # CLI for infrastructure provisioning (Pulumi-based)
│   ├── commands/   # up, down, status, config, pool (add/remove/list)
│   ├── infra/      # Pulumi programs, stack management, state module, post-deploy steps
│   └── options.py  # Shared --project/--zone/--cluster Click options (common_options decorator)
├── credentials.py  # Credential verification & auto-setup (shared by core & CLI)
└── constants.py    # Zone/region utilities, get_default_cluster_name()
```

## Execution Pipeline

```python
@kinetic.run() called
  → JobContext.from_params()        # Resolve config from args/env vars
  → ensure_credentials()            # Verify/auto-configure gcloud, ADC, kubeconfig
  → _prepare_artifacts()            # Upload Data, serialize function, zip working dir
  → _build_container()              # Build or retrieve cached Docker image
  → _upload_artifacts()             # Upload payload.pkl, context.zip to GCS
  → backend.submit_job()            # Create K8s Job (GKE) or LeaderWorkerSet (Pathways)
  → handle.result()                 # Poll until completion, stream logs, download result
```

## Key Modules

| Module                       | Responsibility                                                                                            |
| ---------------------------- | --------------------------------------------------------------------------------------------------------- |
| `core/core.py`               | `@run()` decorator, backend routing, env var capture                                                      |
| `core/accelerators.py`       | Accelerator registry (`GPUS`, `TPUS`), parser (`parse_accelerator`)                                       |
| `credentials.py`             | Credential verification & auto-setup (gcloud, ADC, kubeconfig)                                            |
| `backend/execution.py`       | `JobContext` dataclass (carries `cluster_name`), `BaseK8sBackend` base class, `submit_remote()` pipeline |
| `backend/gke_client.py`      | K8s Job creation, status polling, pod log retrieval                                                       |
| `backend/pathways_client.py` | LeaderWorkerSet creation for multi-host TPUs                                                              |
| `infra/container_builder.py` | Content-hashed Docker image building via Cloud Build                                                      |
| `data/data.py`               | `Data` class, content hashing, data ref serialization                                                     |
| `utils/packager.py`          | `save_payload()` (cloudpickle), `zip_working_dir()`, Data ref extraction                                  |
| `utils/storage.py`           | GCS upload/download/cleanup for job artifacts and Data cache                                              |
| `runner/remote_runner.py`    | Runs inside container: resolve Data refs/volumes, execute, upload result                                  |
| `cli/infra/state.py`         | Centralized Pulumi state: `load_state()`, `apply_update()`, `apply_destroy()`                             |
| `cli/options.py`             | Shared `common_options` Click decorator (`--project`/`--zone`/`--cluster`)                                |
| `cli/commands/pool.py`       | Node pool add/remove/list commands                                                                        |
| `cli/infra/post_deploy.py`   | kubectl, LWS CRD, GPU driver setup after stack.up()                                                       |
| `cli/constants.py`           | CLI defaults, paths, API list                                                                             |
| `cli/main.py`                | CLI entry point (`kinetic` command)                                                                  |

## Key Abstractions

- **`JobContext`** (`backend/execution.py`): Mutable dataclass carrying all job state through the pipeline — inputs, generated IDs, artifact paths, image URI, `cluster_name` (for cluster-scoped bucket/repo resolution).
- **`BaseK8sBackend`** (`backend/execution.py`): Base class with `submit_job`, `wait_for_job`, `cleanup_job`. Subclassed by `GKEBackend` and `PathwaysBackend`.
- **`GpuConfig` / `TpuConfig`** (`core/accelerators.py`): Frozen dataclasses for accelerator metadata. Single source of truth used by runtime, container builder, and CLI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keras-team/kinetic](https://github.com/keras-team/kinetic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
