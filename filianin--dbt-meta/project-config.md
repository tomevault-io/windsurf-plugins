---
trigger: always_on
description: Validates compiled SQL against BigQuery without executing.
---

# CLAUDE.md

## Project Overview

**dbt-meta** is an AI-first CLI tool for extracting metadata from dbt's `manifest.json`.

**Key Design Principles:**
- Performance-first: LRU caching, orjson parser, lazy loading
- AI-optimized: JSON output mode, deterministic responses
- Production-first: Automatically prioritizes production manifest
- Fallback-enabled: BigQuery fallback when models missing from manifest

## Development Setup

```bash
# Install in development mode
pip install -e ".[dev]"

# Run tests (95%+ coverage required)
pytest --cov=dbt_meta

# Type checking + linting
mypy src/dbt_meta && ruff check src/dbt_meta
```

## Development Guidelines

### File Management

- **NO temporary files in `/tmp/`** - Save all files in project root instead
- Temporary files should be visible in git (easy to review and discard)
- Test scripts, debug files, analysis files - all go in project root
- Example: `./test_catalog_fallback.sh` instead of `/tmp/test_catalog_fallback.sh`

## Architecture

### Module Structure

```
src/dbt_meta/
├── cli.py                # Typer CLI + Rich formatting
├── commands.py           # Command implementations + BigQuery fallback
├── errors.py             # Exception hierarchy
├── config.py             # Configuration management
├── fallback.py           # 3-level fallback strategy
├── utils/                # Utility modules
│   ├── __init__.py       # Parser caching, warnings
│   └── git.py            # Git operations
└── manifest/
    ├── parser.py         # Fast manifest parsing (orjson + caching)
    └── finder.py         # 4-level manifest discovery
```

### Key Patterns

#### 1. Three-Level Caching Strategy

```python
# Level 1: Parser instance caching (commands.py:20-34)
@lru_cache(maxsize=1)
def _get_cached_parser(manifest_path: str) -> ManifestParser

# Level 2: Manifest lazy loading (manifest/parser.py:28-58)
@cached_property
def manifest(self) -> Dict[str, Any]

# Level 3: orjson for fast parsing (6-20x faster than stdlib)
```

**Result:** Sub-10ms response times after first command.

**CRITICAL:** Always use `_get_cached_parser()`, never instantiate `ManifestParser` directly.

#### 2. Manifest Discovery (3-level priority)

```
1. --manifest PATH (explicit CLI flag - highest priority)
2. DBT_DEV_MANIFEST_PATH (when --dev flag used, default: ./target/manifest.json)
3. DBT_PROD_MANIFEST_PATH (production, default: ~/dbt-state/manifest.json)
```

**Critical distinction:**
- Production manifest uses `config.alias` for table names
- Dev manifest uses SQL filename for table names
- When both `--manifest` and `--dev` are used, `--dev` is ignored with a warning
- Always use `--dev` flag for dev tables

#### 3. BigQuery Fallback Pattern

```python
model = parser.get_model(model_name)
if not model:
    if os.environ.get('DBT_FALLBACK_BIGQUERY', 'true').lower() in ('true', '1', 'yes'):
        dataset, table = _infer_table_parts(model_name)
        bq_metadata = _fetch_table_metadata_from_bigquery(dataset, table)
        # Return partial metadata with warning to stderr
```

**Supported:** `schema`, `columns`, `info`, `config`
**Not supported:** `deps`, `sql`, `parents`, `children` (dbt-specific)

#### 3a. Catalog Staleness Logic

For `meta columns`, catalog.json is used as a fast alternative to BigQuery (~10ms vs ~3s).

**Staleness detection uses FILE mtime, not internal generated_at:**

```python
# File mtime > 24h → fallback to BigQuery (CI/CD might be broken)
file_age = parser.get_file_age_hours()
if file_age > 24:
    return fallback_to_bigquery()

# Internal generated_at > 7 days → info message only (no fallback)
internal_age = parser.get_age_hours()
if internal_age > 168:
    print(f"ℹ️  Catalog was generated {days}d {hours}h ago")
```

**Why this design:**
- Catalog is synced from CI/CD on each merge to master
- File mtime indicates when sync happened (fresh = CI working)
- Internal `generated_at` can be old if no schema changes occurred
- Old internal age is not a problem if file is regularly synced

Location: `catalog/parser.py:201-217`, `command_impl/columns.py:270-282`

#### 3b. BigQuery Fallback Schema Resolution

**CRITICAL FIX (v0.1.3):** BigQuery fallback now correctly uses production schema for `MODIFIED_UNCOMMITTED` models.

**Problem solved:**
```
# Before fix (WRONG):
Failed to fetch from: personal_pavel_filianin.stg_google_play__installs_app_version

# After fix (CORRECT):
Fetching from: staging_google_play.installs_app_version
```

**Schema resolution logic in `_fetch_from_bigquery_with_model()`:**

```python
if self.use_dev:
    # Dev mode: use model's schema (from dev manifest)
    schema = model.get('schema', '')
    table = self.model_name  # Full model name
else:
    # Production mode: ALWAYS use prod_model for schema
    # Even if model came from dev manifest fallback
    source_model = prod_model if prod_model else model
    schema = source_model.get('schema', '')
    table = source_model.get('alias') or source_model.get('name', '')
```

**Key principle:**
- `prod_model` is fetched from production manifest during state detection
- Passed to fallback methods for correct schema resolution
- For `MODIFIED_UNCOMMITTED` without `--dev`: uses production schema
- For `NEW_*` states: uses dev schema (correct - they only exist in dev)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Filianin/dbt-meta](https://github.com/Filianin/dbt-meta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
