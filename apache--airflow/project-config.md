---
trigger: always_on
description: <!-- SPDX-License-Identifier: Apache-2.0
---

 <!-- SPDX-License-Identifier: Apache-2.0
      https://www.apache.org/licenses/LICENSE-2.0 -->

# AGENTS instructions

## Naming

Write **Dag** (title case) in all prose. Keep the all-caps or lowercase
spelling only when reproducing a literal code token — never rewrite these,
even inside fenced code blocks:

- Python: the SDK class `DAG` (`from airflow.sdk import DAG`,
  `dag = DAG("my_dag", ...)`); identifiers like `dag_id`, `dag`, `my_dag`.
- CLI: `airflow dags list`, `airflow dags test`, etc.
- Paths and config keys: `dag_processing/`, `dagprocessor`, `get_dag`, etc.
- Anti-pattern quotes that show the wrong form to teach the rule itself
  (e.g., `Use "DAG" — always write "Dag"`).

Don't spell out **Directed Acyclic Graph** except for historical context.

## Environment Setup

- Install prek: `uv tool install prek`
- Enable commit hooks: `prek install`
- Install breeze shim (one-time, per machine): `scripts/tools/setup_breeze` — installs `~/.local/bin/breeze` that runs breeze via `uvx` from the current git worktree's `dev/breeze` (so each worktree, including ephemeral agent worktrees, gets its own breeze tied to its sources). See [ADR 0017](dev/breeze/doc/adr/0017-use-uvx-to-run-breeze-from-local-sources.md).
- **Never run pytest, python, or airflow commands directly on the host** — always use `breeze`.
- Place temporary scripts in `dev/` (mounted as `/opt/airflow/dev/` inside Breeze).

## Commands

`<PROJECT>` is folder where pyproject.toml of the package you want to test is located. For example, `airflow-core` or `providers/amazon`.
`<target_branch>` is the branch the PR will be merged into — usually `main`, but could be `v3-1-test` when creating a PR for the 3.1 branch.

<!-- START generated-commands, please keep comment here to allow auto update -->
- **Run a single test:** `uv run --project <PROJECT> pytest path/to/test.py::TestClass::test_method -xvs`
- **Run a test file:** `uv run --project <PROJECT> pytest path/to/test.py -xvs`
- **Run all tests in package:** `uv run --project <PROJECT> pytest path/to/package -xvs`
- **If uv tests fail with missing system dependencies, run the tests with breeze**: `breeze run pytest <tests> -xvs`
- **Run a Python script:** `uv run --project <PROJECT> python dev/my_script.py`
- **Run core or provider tests suite in parallel:** `breeze testing <test_group> --run-in-parallel` (test groups: `core-tests`, `providers-tests`)
- **Run core or provider db tests suite in parallel:** `breeze testing <test_group> --run-db-tests-only --run-in-parallel` (test groups: `core-tests`, `providers-tests`)
- **Run core or provider non-db tests suite in parallel:** `breeze testing <test_group> --skip-db-tests --use-xdist` (test groups: `core-tests`, `providers-tests`)
- **Run single provider complete test suite:** `breeze testing providers-tests --test-type "Providers[PROVIDERS_LIST]"` (e.g., `Providers[google]` or `Providers[amazon]` or "Providers[amazon,google]")
- **Run Helm tests in parallel with xdist** `breeze testing helm-tests --use-xdist`
- **Run Helm tests with specific K8s version:** `breeze testing helm-tests --use-xdist --kubernetes-version 1.35.0`
- **Run specific Helm test type:** `breeze testing helm-tests --use-xdist --test-type <type>` (types: `airflow_aux`, `airflow_core`, `apiserver`, `dagprocessor`, `other`, `redis`, `security`, `statsd`, `webserver`)
- **Run other suites of tests** `breeze testing <test_group>` (test groups: `airflow-ctl-tests`, `docker-compose-tests`, `task-sdk-tests`)
- **Run scripts tests:** `uv run --project scripts pytest scripts/tests/ -xvs`
- **Run Airflow CLI:** `breeze run airflow dags list`
- **Type-check (non-providers):** run the prek hook — `prek run mypy-<project> --all-files` (e.g. `mypy-airflow-core`, `mypy-task-sdk`, `mypy-shared-logging`; each `shared/<dist>` workspace member has its own `mypy-shared-<dist>` hook). The hook uses a dedicated virtualenv and mypy cache under `.build/mypy-venvs/<hook>/` and `.build/mypy-caches/<hook>/`; mypy itself is installed from `uv.lock` via the `mypy` dependency group (`uv sync --group mypy`), so it never mutates your project `.venv`. The hook prefers `uv` from the project's main `.venv/bin/uv` (installed by `uv sync` — `uv` is part of the `dev` dependency group via the `all` extras) for a project-pinned uv version; it falls back to `uv` on `$PATH` with a warning if that binary is missing. Clear with `breeze down --cleanup-mypy-cache`.
- **Type-check (providers):** `breeze run mypy path/to/code`
- **Lint with ruff only:** `prek run ruff --from-ref <target_branch>`
- **Format with ruff only:** `prek run ruff-format --from-ref <target_branch>`
- **Run regular (fast) static checks:** `prek run --from-ref <target_branch> --stage pre-commit`
- **Run manual (slower) checks:** `prek run --from-ref <target_branch> --stage manual`
- **Build docs:** `breeze build-docs`
- **Determine which tests to run based on changed files:** `breeze ci selective-check --commit-ref <commit_with_squashed_changes>`
<!-- END generated-commands, please keep comment here to allow auto update -->

SQLite is the default backend. Use `--backend postgres` or `--backend mysql` for integration tests that need those databases. If Docker networking fails, run `docker network prune`.

## Repository Structure

UV workspace monorepo. Key paths:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/airflow](https://github.com/apache/airflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
