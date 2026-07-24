---
trigger: always_on
description: - Do **NOT** add flowery comments in the code all over the place. The code is self-descriptive, **ONLY** add comments when the code is doing something non-orthodox
---

# Copilot Instructions — dbt-fabricspark

## Comments in code

- Do **NOT** add flowery comments in the code all over the place. The code is self-descriptive, **ONLY** add comments when the code is doing something non-orthodox
- Do **NOT** indirectly refer to Github issues in the actual code. The codebase has nothing to do with Github, it's self-contained.

> 🛑 DO NOT ADD UNNECESSARY COMMENTS IN THE CODE 🛑

## What this repo is

A **dbt adapter** that lets dbt-core build SQL models against Apache Spark in Microsoft Fabric. 

It connects to Fabric Lakehouses via the Livy API (or a local Livy in the devcontainer for offline work) and ships materializations for table, view, incremental, seed, snapshot, and `materialized_lake_view`.

Auto-detects schema-enabled vs non-schema lakehouses (four-part/three-part vs two-part naming) and supports cross-lakehouse writes via `database` overrides on a single profile.

## Key paths

| Path                                                                 | What lives there                                                                                               |
| -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `src/dbt/adapters/fabricspark/`                                      | Python adapter — `impl.py`, `connections.py`, `livysession.py`, `credentials.py`, `mlv_api.py`, `shortcuts.py` |
| `src/dbt/include/fabricspark/macros/`                                | Jinja SQL macros (`adapters/`, `materializations/{models,seeds,snapshots}`, `utils/`)                          |
| `src/dbt/include/fabricspark/{dbt_project.yml,profile_template.yml}` | Adapter package + profile prompts                                                                              |
| `tests/unit/`                                                        | Pure pytest unit tests — no Fabric needed                                                                      |
| `tests/functional/`                                                  | Live-Fabric functional tests, `orchestrator.py`, YAML scheduler, `test_config.yaml`                            |
| `tests/fixtures/dbt-jaffle-shop/`                                    | Project used by the local end-to-end test                                                                      |
| `tools/scripts/run.sh`                                               | Single entry point that every Nx target shells out to                                                          |
| `tools/gh-automation/`                                               | TypeScript helpers for managing Copilot-driven PRs                                                             |
| `docker/Compose.sqlserver.metastore.yaml`                            | SQL Server Hive metastore for the local-e2e flow                                                               |
| `.devcontainer/`                                                     | Prebuilt devcontainer image (pinned by digest) used locally and in CI                                          |
| `.github/workflows/ci.yml`                                           | The CI pipeline                                                                                                |
| `project.json` / `nx.json`                                           | Nx target definitions — the source of truth for build/lint/test                                                |
| `pyproject.toml`                                                     | Python deps (`uv`-managed), ruff config, pytest config                                                         |
| `CHANGELOG.md` / `docs/RELEASE_CHECKLIST.md`                         | Manual changelog + release flow                                                                                |

## CI is the contract — run it locally with `npx nx`

CI (`.github/workflows/ci.yml`) runs **inside the prebuilt devcontainer image** and invokes `npx nx affected ... -t {build|lint|test} --configuration=ci`, followed by `npm run fail-on-untracked-files`. Everything Nx runs delegates to `tools/scripts/run.sh <target>`, which auto-creates a `.venv` via `uv` — never call `pip` directly.

The Nx targets defined in `project.json` (run them with `npx nx run dbt-fabricspark:<target>`):

| Nx target           | Command                                      | What it does                                                                                       |
| ------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `lint`              | `npx nx run dbt-fabricspark:lint`            | `ruff check --fix` + `ruff format` + verify clean                                                  |
| `build`             | `npx nx run dbt-fabricspark:build`           | `uv build` wheel + `twine check`                                                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/dbt-fabricspark](https://github.com/microsoft/dbt-fabricspark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
