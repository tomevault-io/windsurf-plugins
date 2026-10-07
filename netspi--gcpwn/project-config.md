---
trigger: always_on
description: Guidance for AI assistants (and humans) working in this repo. Read this before
---

# CLAUDE.md

Guidance for AI assistants (and humans) working in this repo. Read this before
making changes — it captures the module contract and a few non-obvious
invariants that are easy to violate.

## What this is

GCPwn is a Pacu-style interactive GCP offensive-security framework. A REPL loads
credentials into a workspace, then runs per-service **modules** that enumerate
resources, run exploits, and feed a SQLite-backed data model used for
permission analysis and BloodHound-style attack-path graphing (OpenGraph).

## Layout

- `gcpwn/cli/` — REPL, command parsing, and module dispatch.
  - `main.py` — entrypoint (`python -m gcpwn` / `gcpwn`), workspace selection,
    and `--module` passthrough for unauthenticated modules.
  - `workspace_instructions.py` — the `CommandProcessor` REPL (creds/projects/
    modules/data/configs commands, readline completion).
  - `module_actions.py` — `interact_with_module()`: auth gating, project-scope
    planning, and per-project execution of a module's `run_module`.
- `gcpwn/core/` — framework primitives.
  - `session.py` — `SessionUtility`: credentials, project context, and the
    `get_data`/`insert_data`/`insert_actions` data API modules call.
  - `db.py` — `DataController`: one unified SQLite DB (`databases/gcpwn.db`)
    and all SQL. `workspaces` is the parent table; `session`/`session_actions`
    and every service table carry a `workspaces(id)` FK with `ON DELETE CASCADE`
    (one file so FKs can span the groups; `foreign_keys=ON`).
  - `utils/service_runtime.py` — shared module helpers (arg parsing, error handling,
    paging, `parallel_map`).
  - `action_schema.py` — the permission/provenance column model.
- `gcpwn/modules/` — module tree, grouped by domain:
  - `gcp/<service>/<category>/<module>.py` — GCP service modules. Categories:
    `enumeration`, `exploit`, `unauthenticated`, `process`, `utilities`.
    Per-service `utilities/helpers.py` holds the real API logic; the module
    file is a thin CLI wrapper.
  - `workspace/` — Google Workspace / Cloud Identity modules.
  - `everything/` — cross-cutting orchestrators (`enum_all`, `enum_gcp`,
    `enum_gcp_policy_bindings`, `exploit_gcp_setiampolicy`,
    `process_gcp_iam_bindings`).
  - `opengraph/` — BloodHound OpenGraph build/export and text attack-path
    analysis (`process_og_gcpwn_data`, `process_og_attack_paths`, etc.).
- `gcpwn/mappings/*.json` — module registry + static IAM/escalation data.
- `databases/` — runtime SQLite stores (gitignored). Dev checkout uses
  `databases/gcpwn.db` relative to the repo root; pip-installed builds use
  `~/.gcpwn/databases/gcpwn.db` (or `$GCPWN_HOME/databases/gcpwn.db`).

## Run / test / lint

```bash
python -m gcpwn                                  # run the REPL
python -m pytest -q tests/unit tests/module_contracts   # tests
ruff check .                                     # lint (BLOCKING in CI)
mypy                                             # type-check core (informational)
```

Install dev tooling with `pip install .[dev]` (adds pytest, ruff, mypy).

## The module contract

Every runnable module exposes:

```python
def run_module(user_args, session):
    ...
    return 1   # truthy/0 == success; -1 == failure
```

- `user_args` is a `list[str]` (argparse-style); parse it with the helpers in
  `gcpwn/core/utils/service_runtime.py` (`parse_component_args`, `add_standard_arguments`).
- `session` is the `SessionUtility` (or `PassthroughSession` for unauth runs).
- Some modules take an extra `dependency=`/`callback=` flag used when one module
  calls another; default it to `False`.
- Register new modules in `gcpwn/mappings/module_mappings.json`. The contract
  tests (`tests/module_contracts/`) assert every mapped location exists and
  defines `run_module`, and that no `ArgumentParser(output_format=...)` slips in.
- Module behavior policy (auth required, run-once vs per-project, project flags)
  is decided in `module_actions.get_module_action()` /
  `MODULE_POLICY_REGISTRY`. Unauth modules are detected by an `unauth_` name
  prefix or an `.unauthenticated.` path segment.

## Critical invariants

1. **DB access is serialized by one process-wide lock — concurrent threads are
   safe, but writes still queue.** `DataController` opens its connections with
   `check_same_thread=False` and wraps **every** public method in `@_synchronized`
   against a process-wide re-entrant `RLock` (`db.py`). So calling `session.get_data`
   / `insert_data` / `insert_actions` from a worker thread is *safe* (it blocks on
   the lock; it will **not** raise `sqlite3.ProgrammingError`). `enum_all`'s parallel
   orchestrator and the IAM-bindings pipeline deliberately rely on this — their pool
   workers run whole `run_module`s, saves included. **Nested pools are fine** (the
   `enum_all` outer service pool × a service's inner region/zone fan-out via
   `parallel_map`); peak concurrency ≈ `--threads²` API calls, so watch *GCP rate
   limits*, not the DB.
   Still **prefer the collect-on-main pattern** inside `run_components` and similar
   fan-out: workers do network/CPU work and **return** results, the caller saves.
   It's clearer and avoids piling every worker onto the single write lock (writes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NetSPI/gcpwn](https://github.com/NetSPI/gcpwn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
