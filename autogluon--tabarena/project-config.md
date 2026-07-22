---
trigger: always_on
description: Guidance for coding agents working in `packages/tabflow_slurm/`. Human-facing docs: [`README.md`](README.md).
---

# AGENTS.md — tabflow_slurm

Guidance for coding agents working in `packages/tabflow_slurm/`. Human-facing docs: [`README.md`](README.md).
Repo-wide guidance: [`../../AGENTS.md`](../../AGENTS.md).

## What this is

A package that generates and launches **SLURM** array jobs for TabArena benchmarks. The user
composes a `TabArenaBenchmarkPlan`, calls `setup_jobs()`, and gets `sbatch` command(s); each array
task fits one `(task, fold, repeat, config)` item at a time and caches the result. It is
self-contained and only depends on `tabarena`.

> It **is** a package (`pyproject.toml`, `tabflow_slurm/__init__.py`) and a uv-workspace member,
> installed editable from `packages/tabflow_slurm/`.

## Layout

```
tabflow_slurm/                      ← this folder (docs, examples, history, pyproject)
├── README.md, AGENTS.md, BENCHMARK_LOG.md
├── pyproject.toml                  ← declares the `tabflow_slurm` package; deps: tabarena
├── experiments/                    ← runnable run_*.py scripts (setup + eval subcommands)
└── tabflow_slurm/                  ← the package
    ├── __init__.py                 ← re-exports the public API
    ├── run_tabarena_experiment.py  ← runner: fits ONE item on a node (bundled script)
    ├── submit_template.sh          ← sbatch array script (parses job JSON via jq, calls runner)
    ├── slurm_utils.py              ← setup_slurm_job(): per-node Ray init (caches via JobBatch.cache_config)
    └── setup/                      ← the building blocks
        ├── plan.py                 ← TabArenaBenchmarkPlan (ENTRY POINT), ModelJob, SingleModel
        ├── benchmark.py            ← TabArenaBenchmarkSetup (INTERNAL per-run engine)
        ├── paths.py                ← PathSetup, get_run_script_path/get_submit_script_path
        ├── resources.py            ← ResourcesSetup + v0.1/BeyondArena presets
        └── scheduler.py            ← SchedulerSetup → SlurmSetup → GCPSlurmSetup (batching + sbatch)
```

## The public-API boundary

- **Entry point:** `TabArenaBenchmarkPlan` (in `setup/plan.py`). Everything users touch is
  re-exported from the top-level `tabflow_slurm` package and from `tabflow_slurm.setup`. When adding
  a public symbol, update **both** `__init__.py` `__all__` lists.
- **Internal:** `TabArenaBenchmarkSetup` (`setup/benchmark.py`) is the per-run engine. The plan
  builds/drives it (one per group). Don't push users toward it directly.

## Mental model of `setup_jobs()`

1. `plan.setup_jobs()` → `_prefetch_model_weights()` (head node) → `build_setups()`.
2. `build_setups()` → `_group_jobs()`: each `ModelJob`'s `resources`/`scheduler`/`experiment` dict
   overrides are applied to the base building blocks via `dataclasses.replace`; its `tasks` (a
   `TaskSubset`) is merged onto the plan's `task_subset` (job wins per field). Jobs are **merged**
   when their `(resources, scheduler, task_subset, experiment-with-models-zeroed, ignore_cache)`
   signature matches (compared by value). One `TabArenaBenchmarkSetup` per group.
3. Each setup's `get_jobs_to_run()`: ensure dirs → `experiment_bundle.build_experiments()`
   (attaches each experiment's `ModelConstraints`) → `context.build_jobs(experiments,
   task_subset=...)` (scopes the context's collection by the `TaskSubset`, then enumerates
   experiments × splits; constraint-violating pairs are dropped during enumeration) → scope the
   context's collection to the jobs' tasks and `materialize()` (downloads only those) → Ray cache
   check (core `job_cache_exists_batch` over plain coordinate tuples) → persist the surviving sweep
   as a `JobBatch` artifact (experiments.yaml + task_metadata.csv + jobs.json) →
   `scheduler.bundle_items()`.
4. `scheduler.get_run_commands()` writes the job JSON (splitting at `max_array_size`) and returns the
   `sbatch` command(s). The plan prints one consolidated summary.

Then: `sbatch … submit_template.sh <job.json>` → array task picks `jobs[SLURM_ARRAY_TASK_ID]` → runs
`run_tabarena_experiment.py` per item → `setup_slurm_job()` + `JobBatch.load()` +
`ExperimentBatchRunner.run_jobs()` (the exact same execution path as a local benchmark run).

## Conventions

- Building blocks are **frozen-ish dataclasses**; per-job customization is via override **dicts** on
  `ModelJob` (validated against the dataclass fields — unknown keys raise). Add new knobs as
  dataclass fields, not ad-hoc params.
- **Bundled scripts** (`run_tabarena_experiment.py`, `submit_template.sh`) live at the *package
  root* and are resolved via `get_run_script_path()` / `get_submit_script_path()` (relative to the
  install). `pyproject.toml` ships `*.sh` as package data — keep that if you add/rename shell files.
- The **job JSON** is the contract between Python and `submit_template.sh`: `{"defaults": {...},
  "jobs": [{"items": [{experiment, dataset, fold, repeat}]}]}`. If you change `defaults` keys, update
  the `jq` reads in `submit_template.sh` too. Items are self-describing coordinates: the runner
  resolves `experiment` by name and `dataset` against the shipped `JobBatch` artifact
  (`defaults.job_batch_dir`).
- `benchmark_name` vs `parallel_safe_benchmark_name`: the former is shared (output + log dirs); the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autogluon/tabarena](https://github.com/autogluon/tabarena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
