---
trigger: always_on
description: `dbt-osmosis` is a Python CLI and package for dbt development workflows. The repo centers on four surfaces:
---

# Repository Guidelines

MANDATORY: Use 10x.

## Project Overview

`dbt-osmosis` is a Python CLI and package for dbt development workflows. The repo centers on four surfaces:

- schema YAML management (`yaml organize`, `yaml document`, `yaml refactor`)
- column-level documentation inheritance across dbt lineage
- ad-hoc SQL compile/run helpers
- an optional Streamlit workbench for interactive dbt SQL development

Other CLI families (`diff`, `lint`, `test`, `generate`, `nl`, `test-llm`) reuse the same project/bootstrap spine rather than defining separate runtimes.

Primary entrypoint: `src/dbt_osmosis/cli/main.py`
Package entrypoint: `src/dbt_osmosis/__main__.py`

## Architecture & Data Flow

### Main execution spine
1. Click commands in `src/dbt_osmosis/cli/main.py` parse flags and build `DbtConfiguration`.
2. `src/dbt_osmosis/core/config.py:create_dbt_project_context()` loads the dbt project, adapter, and manifest.
3. YAML commands create `YamlRefactorContext` from `src/dbt_osmosis/core/settings.py`.
4. Candidate nodes are filtered in `src/dbt_osmosis/core/node_filters.py`.
5. Transform chains in `src/dbt_osmosis/core/transforms.py` mutate model/source metadata.
6. YAML is read and written through `src/dbt_osmosis/core/schema/reader.py` and `writer.py`.
7. `src/dbt_osmosis/core/sync_operations.py` merges manifest-backed truth back into schema files.

### Key architectural boundaries
- `src/dbt_osmosis/core/introspection.py` is the configuration and property-resolution center. Prefer `SettingsResolver` and `PropertyAccessor` over ad hoc config lookups.
- `src/dbt_osmosis/core/path_management.py` owns YAML routing and project-root safety checks.
- `src/dbt_osmosis/core/inheritance.py` builds the column knowledge graph used for documentation inheritance.
- `src/dbt_osmosis/core/sql_operations.py` is the shared SQL compile/execute path used by CLI, workbench, and proxy code.
- `src/dbt_osmosis/core/schema/parser.py`, `reader.py`, and `writer.py` split YAML concerns deliberately: filter dbt-osmosis-owned sections, cache reads, then restore preserved sections on atomic write.
- `src/dbt_osmosis/workbench/app.py` reuses the same dbt context but owns Streamlit state and dashboard composition.

### Public vs. internal surfaces
- `src/dbt_osmosis/core/osmosis.py` is the compatibility/public facade. `src/dbt_osmosis/core/__init__.py` is no longer a re-export surface; internal code should import concrete submodules directly.
- Deep edits under `src/dbt_osmosis/core/` must also follow `src/dbt_osmosis/core/AGENTS.md`.

## Key Directories

- `src/dbt_osmosis/cli/` — Click command groups and user-facing entrypoints
- `src/dbt_osmosis/core/` — dbt context setup, config resolution, transforms, YAML I/O, inheritance, plugins
- `src/dbt_osmosis/core/schema/` — round-trip YAML parsing, caching, writing, validation
- `src/dbt_osmosis/sql/` — SQL proxy and related helpers
- `src/dbt_osmosis/workbench/` — Streamlit workbench and dashboard components
- `tests/` — pytest suite; `tests/core/` mirrors core modules, root tests cover higher-level YAML behavior
- `demo_duckdb/` — canonical dbt fixture project used by tests and examples
- `docs/` — Docusaurus docs site; actual content lives under `docs/docs/`
- `specs/001-unified-config-resolution/` — detailed spec/plan/quickstart for config-resolution work
- `_deps/` — vendored dbt packages; avoid editing unless the task explicitly targets vendored code

## Important Files

| Path | Why it matters |
| --- | --- |
| `pyproject.toml` | Source of truth for Python support, dependencies, console script, Ruff, pytest, pyright |
| `Taskfile.yml` | Canonical developer workflow (`task format`, `task lint`, `task test`, `task dev`) |
| `.pre-commit-config.yaml` / `.pre-commit-hooks.yaml` | Repo hygiene policy plus packaged `dbt-osmosis yaml refactor -C` pre-commit hook contract |
| `src/dbt_osmosis/cli/main.py` | Complete CLI surface: `yaml`, `sql`, `workbench`, `generate`, `nl`, `test`, `test-llm`, `lint`, `diff` |
| `docs/package.json` / `docs/docusaurus.config.js` | Source of truth for docs-site tooling and Docusaurus 3 configuration |
| `demo_duckdb/dbt_project.yml` / `demo_duckdb/dbt-osmosis.yml` | Best concrete examples of routing rules, config precedence, and YAML formatting defaults |
| `src/dbt_osmosis/core/config.py` | dbt project/bootstrap and manifest loading |
| `src/dbt_osmosis/core/settings.py` | `YamlRefactorContext`, formatter settings, catalog handling |
| `src/dbt_osmosis/core/introspection.py` | `SettingsResolver`, `PropertyAccessor`, caches, config precedence |
| `src/dbt_osmosis/core/schema/parser.py` / `reader.py` / `writer.py` | Canonical round-trip YAML filter/cache/preserve pipeline |
| `src/dbt_osmosis/core/transforms.py` | `TransformPipeline` and main YAML mutation operations |
| `src/dbt_osmosis/core/inheritance.py` | column lineage and inheritance logic |
| `src/dbt_osmosis/core/sql_operations.py` | Shared SQL compile/execute helpers used outside just the CLI |
| `src/dbt_osmosis/core/path_management.py` | YAML routing, source YAML bootstrapping, root-path validation |
| `src/dbt_osmosis/workbench/app.py` | Streamlit workbench bootstrap and state initialization |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [z3z1ma/dbt-osmosis](https://github.com/z3z1ma/dbt-osmosis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
