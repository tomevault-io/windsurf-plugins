---
trigger: always_on
description: This directory holds the test projects and the automated test harness for the
---

# AGENTS.md — integration_tests

This directory holds the test projects and the automated test harness for the
`dbt_constraints` dbt package. Read this file before you edit or run the tests.

## Layout

```
integration_tests/
├── dbt-legacy-syntax/        # older generic test syntax
│   ├── data/tpc_h_seeds/     # TPC-H seed CSVs
│   ├── macros/               # clone_table + issue_105 custom naming macros
│   ├── models/               # dim_*, fact_* models, sources.yml, schema.yml
│   ├── tests/                # singlular_test.sql, assert_source_constraints.sql
│   ├── packages.yml          # dbt_utils + local dbt_constraints
│   ├── profiles.yml          # local dbt reads this
│   ├── dbt_projects_profiles.yml  # dbt Projects on Snowflake reads this instead
│   └── env.yml               # supplies the values that file reads
├── dbt-current-syntax/       # current `arguments:` test syntax, most cells run this
│   └── (mirrors dbt-legacy-syntax layout, plus tests/assert_fk_parity.sql)
└── automated_tests/          # pytest harness
    ├── conftest.py           # fixtures, matrix parametrization, dbt runner
    ├── dbt_venv.py           # per-cell uv virtual environments
    ├── dpos_stage.py         # builds a deployable copy for the in-database cells
    ├── dpos_runner.py        # translates dbt commands into snow CLI calls
    ├── config/test-versions.json  # dbt version matrix (single source of truth)
    ├── docker/compose/       # DB server containers (postgres/oracle/sqlserver)
    ├── scripts/run_dbt_tests.sh   # full seed + build workflow
    ├── tests/                # parameterized test modules
    └── pytest.ini
```

## Two projects, chosen by dbt version

The harness picks a project by dbt version, not by adapter. dbt-core added the
`arguments:` property for generic tests in 1.10.5. `get_project_dir` in
`conftest.py` compares the cell version against `CURRENT_SYNTAX_MIN_VERSION`.

- `dbt-legacy-syntax/` runs on every engine before dbt-core 1.10.5. Only the
  `snowflake` cell (dbt-core 1.5.12) uses it today. Do not modernise it. It is the
  only proof that the package still supports older dbt.
- `dbt-current-syntax/` runs on dbt-core 1.10.5 and later, on the dbt v2 engine
  targets `fusion` and `core2`, and on both in-database cells. That covers
  `postgres`, `oracle`, `dpos_core`, and `dpos_fusion`. Keep every model and macro
  in it portable across PostgreSQL, Oracle, and Snowflake.

Keep the two source trees in sync. Add new coverage to both. Only the `schema.yml`
files, `models/sources.yml`, `data/tpc_h_seeds/seeds.yml`, and three keys in
`dbt_project.yml` differ. `dbt-current-syntax` wraps test arguments in an
`arguments:` key, sets `always_create_constraint` inside `+meta:`, and sets the
`require_generic_test_arguments_property` flag. `dbt-legacy-syntax` does none of
those.

Both in-database cells stage from `dbt-current-syntax`, so its `env.yml` declares
one environment per cell. `stage_project` sets `default_environment` to the cell
name at stage time, which keeps each cell in its own schema.

## Two execution modes

A cell runs dbt in one of two places. `conftest.py` dispatches on the database key.

| Mode | Keys | Where dbt runs |
|---|---|---|
| Host | `postgres` `oracle` `snowflake` `fusion` `core2` | a cached uv venv on this machine |
| In-database | `dpos_core` `dpos_fusion` | inside Snowflake, through dbt Projects on Snowflake |

Both modes return a `subprocess.CompletedProcess`, because the in-database mode
shells out to the `snow` CLI. The assertions in the tests are identical.

## Harness architecture (current)

- Database servers run as Docker containers, one per adapter, at session scope.
  Compose files live in `docker/compose/`.
- The dbt client runs on the host, not in a container. Each matrix cell
  (database + dbt version) gets its own cached uv venv, created and reused by
  `dbt_venv.py`. This avoids distribution shadowing from a shared base image.
- `config/test-versions.json` is the single source of truth for the matrix.
  `conftest.py` reads `dbt_versions` there to parametrize tests.
- The `adapters` block in that file is descriptive only. Test code ignores it.
- Legacy files `.dockerenv/`, `docker-compose.test-runner.yml`, and
  `requirements-test.txt` runner build steps are not part of the active path.

## In-database cells (dbt Projects on Snowflake)

The `dpos_core` and `dpos_fusion` cells deploy the project to Snowflake and run every
command there. Snowflake supplies the engine, so the host installs no dbt client for the
run. It still builds one venv per cell to run `dbt deps` while staging.

Each test session:

1. `dpos_stage.py` copies the project to `automated_tests/.dpos-stage/<cell>/`, copies
   the repository root to `<stage>/local_packages/dbt_constraints/`, rewrites
   `packages.yml`, runs `dbt deps`, and prunes each installed package's own test suite.
2. `dpos_runner.py` runs `snow dbt deploy`, which adds a new `VERSION$N`.
3. Each test command becomes `snow dbt execute ... --dbt-version <version>`.

Rules that follow from the platform, not from choice:

- **`- local: ../../` cannot be deployed.** Snowflake rejects a package path that leaves
  the project root. The staging copy exists only for this.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Snowflake-Labs/dbt_constraints](https://github.com/Snowflake-Labs/dbt_constraints) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
