---
trigger: always_on
description: Guidance for working in the **d6tflow** repo.
---

# CLAUDE.md

Guidance for working in the **d6tflow** repo.

## What this is

d6tflow is a small, self-contained Python library for building data-science workflows: you
declare `Task` classes with parameters, dependencies (`requires`), and a `run()` that
`save()`s output; the engine runs the DAG in dependency order and skips tasks whose output
already exists. It has no heavyweight workflow-engine dependency — the task model, parameters,
and executor all live in `d6tflow/core.py` + `d6tflow/parameter.py`.

## Layout

```
d6tflow/
  __init__.py        # public API: run, preview, Workflow, WorkflowMulti, FlowExport/Import,
                     #   invalidate_*, requires/inherits, re-exported Parameter types
  core.py            # the engine: Task, Register metaclass, Target/LocalTarget, flatten,
                     #   getpaths, task_id_str, inherits/requires, find_deps, build()
  parameter.py       # Parameter, Int/Float/Bool/Date/Dict/List/Enum Parameter
  tasks/__init__.py  # TaskData (+ TaskCache/Json/Pickle/CSV/Excel/Pq/Markdown...), TaskAggregator
  targets/__init__.py# CacheTarget, _LocalPathTarget, DataTarget + format targets, Target re-export
  settings.py        # global settings (dirpath, cached, check_dependencies, task-id lengths...)
  cache.py           # `data` = in-memory target cache (dict)
  utils.py           # print_tree (preview), traverse, param generators, bcolors
  functional.py      # decorator-based functional Workflow API (uses only d6tflow.requires)
tests/               # pytest suite (see tests/setup.md)
docs/                # examples + docs/todo/ design notes
```

Ignore `bak/`, `build/`, `dist/`, `*.egg-info/`, `data/`, `models/`, `tests-data/` — local
artifacts / backups, not source. (`d6tflow/core/` is an empty stale dir; the real module is
`d6tflow/core.py`.)

## Public API (what exists — check here before adding new code)

Everything below is exported from `d6tflow` (top-level) unless noted. Reach for these
instead of reinventing them.

**Run / load**
- `run(tasks, forced=None, forced_all=False, forced_all_upstream=False, confirm=False, abort=True, ...)` — run task(s) in dependency order (`workers` is accepted but ignored — sequential engine).
- `preview(tasks, show_params=True, ...)` / `show(task)` — print the execution tree without running.
- `runLoad(task, params=None, load=True, taskLoad=None, reset=False)` — one-liner: build a `Workflow`, optionally `reset`, `run`, then `outputLoad` and return it. `runIt(...)` is `runLoad(..., load=False)`. `runIterConcat(...)` is the same shape (experimental). Prefer these for quick run-and-fetch over hand-rolling a `Workflow`.

**Workflow objects** — `Workflow(task=None, params=None, path=None, env=None)` and `WorkflowMulti(...)` (multi-experiment: `params` is `{flow_name: {param: val}}`, methods take an optional `flow=` selector). Key methods on both: `run`, `preview`, `complete`, `outputLoad` / `outputLoadAll` / `outputLoadMeta` / `outputLoadMetaJson`, `outputPath`, `reset` / `reset_upstream` / `reset_downstream`, `set_default`, `get_task`, `attach_flow`.

**Deps / params (decorators)** — `@requires(*tasks | {name: task})` copies parent params **and** wires `requires()`; `@inherits(...)` copies params only (adds `clone_parent`/`clone_parents`, you write `requires()` yourself). Params live in `parameter.py`, re-exported: `Parameter`, `IntParameter`, `FloatParameter`, `BoolParameter`, `DateParameter`, `DictParameter`, `ListParameter`, `EnumParameter` (use `significant=False` to exclude from `task_id`).

**Share / move flows** — `FlowExport(tasks=None, flows=None, save=False, path_export='tasks_export.py')` generates standalone task files; `FlowImport(...)` loads them back. (Generated-text contract — see the FlowExport note under Conventions.)

**Invalidate** — `invalidate_upstream(task, confirm=False)`, `invalidate_downstream(task, task_downstream, confirm=False)`, plus `taskflow_upstream` / `taskflow_downstream`. NB: `invalidate_all` and `invalidate_orphans` are **stubs that raise `NotImplementedError`** — don't point users at them.

**Config** — `set_dir(dir=None)` (init + set data dir), `enable_cloud_storage(protocol, bucket, prefix=None)` / `enable_gcs(bucket, prefix=None)` (fsspec-backed; needs the `gcs`/`s3`/`cloud-base` install extra), `enable_logging()` / `disable_logging()`, and the mutable `settings.*` (`dirpath`, `cached`, `check_dependencies`, `execution_summary`, task-id lengths).

**Functional API** — `d6tflow.functional.Workflow` is a separate decorator-based style (`@flow.task`, `@flow.requires`, `@flow.params`, `@flow.persists`). Independent of the class-based API above; don't mix the two in one example.

**Task-body idiom** (inside `run()`): load upstream with `self.inputLoad()` (single, or `a, b = self.inputLoad()` for multiple; `self.input()[key].load()` only to select one named/indexed input or a specific `persist=` output); save with `self.save(...)` / `self.saveMeta(...)`. Keep examples on `self.inputLoad()` — the docs standardize on it.

## Architecture notes that bite

Read `docs/todo/20260606-sys-decouple-luigi.md` and
`docs/todo/20260606-sys-param-global.md` before changing the engine — they capture the
non-obvious decisions. Highlights:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [d6t/d6tflow](https://github.com/d6t/d6tflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
