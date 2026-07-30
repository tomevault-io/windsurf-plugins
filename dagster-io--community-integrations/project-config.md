---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

dagster-evidence is a Dagster integration for [Evidence.dev](https://evidence.dev/), enabling orchestration of Evidence dashboard projects as Dagster assets. It automatically discovers sources from Evidence projects, generates corresponding Dagster assets, and handles build/deployment pipelines.

## Development Commands

```bash
make install   # Install dependencies (uv sync)
make build     # Build package (uv build)
make test      # Run all tests (uv run pytest)
make ruff      # Format and lint (ruff check --fix && ruff format)
make check     # Type checking (uv run ty check)
make quality   # Run all checks (ruff, format, ty, pytest)

# Run specific tests
uv run pytest dagster_evidence_tests/test_component.py
uv run pytest dagster_evidence_tests/test_translator.py::test_custom_translator
```

**IMPORTANT**: Always run `make quality` after making changes to ensure all checks pass before committing.

## Architecture

### Core Component Flow

```
EvidenceProjectComponentV2 (StateBackedComponent)
    │
    ├── BaseEvidenceProject (LocalEvidenceProject | EvidenceStudioProject)
    │       │
    │       ├── parse_evidence_project() → EvidenceProjectData
    │       │       └── Reads sources/ folder, parses connection.yaml + SQL files
    │       │
    │       └── load_evidence_project_assets() → (AssetsDefinition[], SensorDefinition[])
    │               └── Uses DagsterEvidenceTranslator to convert sources to assets
    │
    └── DagsterEvidenceTranslator
            │
            ├── SOURCE_TYPE_REGISTRY: maps source types to source classes
            │       duckdb → DuckdbEvidenceProjectSource
            │       motherduck → MotherDuckEvidenceProjectSource
            │       bigquery → BigQueryEvidenceProjectSource
            │       gsheets → GSheetsEvidenceProjectSource
            │
            └── get_asset_spec(data) → AssetSpec | AssetsDefinition
                    └── Delegates to source class for source assets
```

### Key Classes

| Class | Location | Purpose |
|-------|----------|---------|
| `EvidenceProjectComponentV2` | `components/evidence_project_v2.py` | Main Dagster component, state-backed |
| `DagsterEvidenceTranslator` | `components/translator.py` | Converts Evidence objects to Dagster assets |
| `BaseEvidenceProjectSource` | `components/sources.py` | Abstract base for data source types |
| `LocalEvidenceProject` | `components/projects.py` | Handles local file-based Evidence projects |
| `BaseEvidenceProjectDeployment` | `components/deployments.py` | Abstract base for deployment targets |

### Source Asset Generation

Each source type (DuckDB, BigQuery, etc.) implements:
1. `extract_data_from_source()` - Parses SQL to extract table dependencies using `sqlglot`
2. `get_source_asset()` - Creates `AssetsDefinition` with automation conditions
3. `get_source_sensor()` - Optional sensor for detecting upstream data changes

### Translator Data Types

- `EvidenceSourceTranslatorData` - Passed to translator for source query assets (contains `source_content`, `source_group`, `query`, `extracted_data`)
- `EvidenceProjectTranslatorData` - Passed to translator for project build asset (contains `project_name`, `sources_by_id`, `source_deps`)

### Deployment Types

- `GithubPagesEvidenceProjectDeployment` - Pushes to GitHub Pages branch using GitPython
- `CustomEvidenceProjectDeployment` - Runs arbitrary shell command
- `EvidenceProjectNetlifyDeployment` - Planned (not yet implemented)

## Extension Points

### Custom Translator
Subclass `DagsterEvidenceTranslator` to customize asset generation:
```python
class MyTranslator(DagsterEvidenceTranslator):
    SOURCE_TYPE_REGISTRY = {
        **DagsterEvidenceTranslator.SOURCE_TYPE_REGISTRY,
        "postgres": PostgresEvidenceProjectSource,
    }

    def get_asset_spec(self, data):
        spec = super().get_asset_spec(data)
        # customize...
        return spec
```

### Adding a New Source Type

To add a new source type (e.g., Postgres, MySQL, Snowflake):

1. **Add the source class** in `components/sources.py`:
   ```python
   @beta
   @public
   class PostgresEvidenceProjectSource(BaseEvidenceProjectSource):
       @staticmethod
       def get_source_type() -> str:
           return "postgres"  # Must match 'type' in connection.yaml

       @classmethod
       def get_hide_source_asset_default(cls) -> bool:
           return True  # Set to True if source assets should be hidden by default

       @classmethod
       def get_source_sensor_enabled_default(cls) -> bool:
           return True  # Set to True to enable change detection sensors

       @classmethod
       def extract_data_from_source(cls, data: "EvidenceSourceTranslatorData") -> dict[str, Any]:
           from dagster_evidence.utils import extract_table_references
           options = data.source_content.connection.extra.get("options", {})
           table_refs = extract_table_references(
               data.query.content,
               default_database=options.get("database"),
               default_schema=options.get("schema", "public"),
           )
           return {"table_deps": table_refs}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dagster-io/community-integrations](https://github.com/dagster-io/community-integrations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
