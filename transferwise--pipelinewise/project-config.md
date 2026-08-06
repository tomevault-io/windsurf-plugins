---
trigger: always_on
description: PipelineWise is an ELT data pipeline framework built on the Singer.io specification. It replicates data from various sources (taps) to analytics data warehouses (targets) with minimal load-time transformations.
---

# AI Coding Agent Instructions

## Purpose
PipelineWise is an ELT data pipeline framework built on the Singer.io specification. It replicates data from various sources (taps) to analytics data warehouses (targets) with minimal load-time transformations.

## Architecture
- **Language:** Python 3.12
- **Framework:** Singer.io (taps and targets communicate via JSON over stdout/stdin)
- **Config format:** YAML (tap/target definitions), JSON (runtime configs, state, catalog)
- **CLI:** argparse-based, entry point at `pipelinewise/cli/__init__.py`. Commands map to methods on `PipelineWise` class via `getattr`. Alias mechanism: command names can be remapped before dispatch (e.g. deprecated `sync_tables` → `fast_sync`; canonical `import_config` and its deprecated `import` alias → `import_project`).

### Key Components
- `pipelinewise/cli/pipelinewise.py` — Main orchestration engine (~2000 lines). `run_tap()` is the core pipeline execution method. `fast_sync()` is the entry point for the `fast_sync` CLI command (aliased from `sync_tables`).
- `pipelinewise/cli/commands.py` — Builds shell commands: `tap | transform-field | mbuffer | target` pipeline.
- `pipelinewise/cli/config.py` — Loads/validates YAML configs, generates runtime JSON files at `~/.pipelinewise/<target_id>/<tap_id>/`.
- `pipelinewise/cli/constants.py` — Connector type enums and mappings.
- `pipelinewise/fastsync/` — Optimized native database-to-database sync (10-100x faster than Singer for full loads).
- `pipelinewise/cli/alert_handlers/` — Slack and VictorOps alerting. Extend by subclassing `BaseAlertHandler`.
- `pipelinewise/cli/schemas/` — JSON Schema files for validating tap/target configs.

### Sync Paths
1. **Singer** — Standard replication via `tap | transform | target` piped processes. Used for ongoing INCREMENTAL and LOG_BASED replication.
2. **FastSync** — Performance optimization that bypasses Singer for bulk data operations using native database tools. Not a replication method — it is an optimization engine with two components:
   - **FullSync** — Exports entire tables and replaces the target. Used automatically for initial syncs and explicitly via the `fast_sync` CLI command. Used for FULL_TABLE replication.
   - **PartialSync** — Exports a filtered range of rows and merges with the target. Used explicitly via the `partial_sync_table` CLI command, or automatically when `fast_sync` encounters tables with `sync_start_from` in the tap config.
   - FastSync infrastructure lives at `pipelinewise/fastsync/`, with shared connectors in `pipelinewise/fastsync/commons/`. PartialSync lives at `pipelinewise/fastsync/partialsync/` and imports from `commons/`.
   - Supported pairs: MySQL/PG/S3-CSV/MongoDB → Snowflake/PG. Defined PartialSync (`sync_start_from`) only: MySQL/PG → Snowflake.

### Supported Connectors
- **Taps:** MySQL, PostgreSQL, MongoDB, Kafka, S3 CSV, Snowflake, Salesforce, Zendesk, Jira, Google Analytics, Oracle, GitHub, Slack, Shopify, Twilio, Zuora, Mixpanel
- **Targets:** Snowflake, PostgreSQL, S3 CSV

## Development Environment
- **Python:** 3.12 required (`python_requires='==3.12.*'`)
- **Setup:** `pip install -e ".[test]"` from repo root
- **Connectors:** Installed separately via `make` targets into `.virtualenvs/` directory

## Build & Test
- **Unit tests:** `pytest tests/` (use `.venv/bin/pytest` if system Python lacks dependencies)
- **Lint:** `flake8`, `pylint`, `ruff`
- **Format check:** `pre-commit run --all-files`
- **Single test:** `pytest tests/path/to/test.py -v`
- **Coverage:** `pytest --cov=pipelinewise tests/`

## Repository Map
- `pipelinewise/cli/` — CLI and orchestration logic
- `pipelinewise/fastsync/` — Optimized sync implementations
- `pipelinewise/fastsync/commons/` — Shared FastSync tap/target connectors (used by both FullSync and PartialSync)
- `pipelinewise/fastsync/partialsync/` — PartialSync implementations (imports from `commons/`)
- `singer-connectors/` — Git submodule references to Singer tap/target repos
- `tests/` — Unit and integration tests
- `docs/` — Documentation
- `dev-project/` — Example project for local development

## Code Style
- Follows PEP 8 with `ruff` and `flake8` enforcement.
- Use `snake_case` for functions/variables, `PascalCase` for classes.
- JSON configs use `snake_case` keys.
- Snowflake identifiers are uppercased in FastSync connectors.

## Documentation Style
- Format: reStructuredText (RST) using Sphinx.
- Heading conventions: `=` for page title (H1), `-` for concept-level pages (H1), `'` for subsections (H3), `"` for sub-subsections (H4). Concept pages (`docs/concept/`) use `-` for titles; user guide pages (`docs/user_guide/`) use `=` for titles.
- RST labels: use `.. _label_name:` before headings for cross-referencing with `:ref:`label_name``.
- Code examples: use `.. code-block:: yaml` (or `bash`, `sql`, etc.), never plain `.. code-block::`.
- Admonitions: `.. warning::`, `.. note::`, `.. tip::`, `.. attention::`, `.. seealso::`.
- When renaming CLI commands, update both the command reference in `cli.rst` and all cross-references (`:ref:` labels, code examples, inline mentions) across all docs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [transferwise/pipelinewise](https://github.com/transferwise/pipelinewise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
