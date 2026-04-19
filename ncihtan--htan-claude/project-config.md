---
trigger: always_on
description: This is a Claude Code skill for working with the **Human Tumor Atlas Network (HTAN)** — an NCI Cancer Moonshot initiative that constructs 3D atlases of the dynamic cellular, morphological, and molecular features of human cancers as they evolve from precancerous lesions to advanced disease. The skill provides tools for accessing HTAN data across four platforms (HTAN Portal ClickHouse, Synapse, CRDC/Gen3, ISB-CGC BigQuery), searching HTAN publications, and querying HTAN metadata.
---

# HTAN Skill for Claude Code

## Project Overview

This is a Claude Code skill for working with the **Human Tumor Atlas Network (HTAN)** — an NCI Cancer Moonshot initiative that constructs 3D atlases of the dynamic cellular, morphological, and molecular features of human cancers as they evolve from precancerous lesions to advanced disease. The skill provides tools for accessing HTAN data across four platforms (HTAN Portal ClickHouse, Synapse, CRDC/Gen3, ISB-CGC BigQuery), searching HTAN publications, and querying HTAN metadata.

## Package Structure

The core functionality lives in the `htan` pip-installable package (`src/htan/`). The skill definition (`skills/htan/SKILL.md`) teaches Claude the CLI commands. No MCP server — just Bash calls to the `htan` CLI.

| Module | Source | Notes |
|---|---|---|
| `htan.config` | `src/htan/config.py` | 3-tier credential resolution (env > keychain > config file) |
| `htan.query.portal` | `src/htan/query/portal.py` | Portal ClickHouse queries, `PortalClient` class |
| `htan.query.bq` | `src/htan/query/bq.py` | BigQuery queries, `BigQueryClient` class |
| `htan.download.synapse` | `src/htan/download/synapse.py` | Synapse downloads |
| `htan.download.gen3` | `src/htan/download/gen3.py` | Gen3/CRDC DRS downloads |
| `htan.pubs` | `src/htan/pubs.py` | PubMed search |
| `htan.model` | `src/htan/model.py` | HTAN data model queries, `DataModel` class |
| `htan.files` | `src/htan/files.py` | File ID to download coordinate mapping |
| `htan.cli` | `src/htan/cli.py` | Unified CLI entry point (`htan` command) |

### What Still Needs Live Testing

- `htan download synapse synXXXXXXXX --dry-run` — test with Synapse credentials
- `htan query bq tables` — test with GCP project
- `htan query bq sql "SELECT ..."` — test actual query execution
- `htan download gen3 resolve` — test with Gen3 credentials (requires dbGaP)

## Environment Setup

The project uses a **uv virtual environment** and a `pyproject.toml`-based package layout.

### uv Package Management Rules

All Python dependencies **must** be managed with `uv`. Never use `pip`, `pip-tools`, `poetry`, or `conda` for dependency tasks.

- **Install the package**: `uv pip install -e ".[dev]"` (editable mode with dev deps)
- **Install with dev deps**: `uv pip install -e ".[dev]"`
- **Run CLI**: `uv run htan <command>` or just `htan <command>` after install
- **Run tests**: `uv run pytest tests/`
- **Run PyPI tools directly**: `uvx ruff`, `uvx pytest`
- **Build wheel**: `uv build`

When executing any Python code in this project, **always use `uv run`** instead of activating the venv manually. This ensures the correct environment is used regardless of shell state.

### Package Dependencies (pyproject.toml)

All platform dependencies are included by default:

```bash
pip install htan              # Everything: portal, Synapse, Gen3, BigQuery, pubs, model
pip install htan[dev]         # + pytest, ruff (for development)
```

## Credential Security

**Important**: Credentials are stored in config files, NOT environment variables in this setup:
- **Portal ClickHouse**: `~/.config/htan-skill/portal.json` (populated by `htan_setup.py init-portal`, downloaded from Synapse project syn73720845 gated by Team:3574960 membership)
- Synapse: `~/.synapseConfig`
- Gen3: `~/.gen3/credentials.json`
- BigQuery: Application Default Credentials (via `gcloud auth application-default login`)

Portal credentials are **not stored in source code**. They are fetched from Synapse behind a team membership gate, providing an audit trail of who accessed them.

**When using Claude Code**, avoid running commands that would print credentials or signed URLs into the conversation (which flows through the Anthropic API). Specifically:
- **Safe to run via Claude**: `--help`, `--dry-run`, PubMed searches, all portal queries (credentials read from local config, not echoed), BigQuery `tables`/`describe`/`sql` (metadata results are not sensitive), file mapping `update`/`lookup`/`stats`, all data model commands (fetches from public GitHub, no credentials)
- **Run in your own terminal**: `htan download gen3 resolve` (outputs signed URLs), any command where error messages might echo tokens

## Project Layout

```
htan-skill/
├── pyproject.toml                    # Package definition (hatchling build)
├── src/
│   └── htan/                         # pip-installable package
│       ├── __init__.py               # Version
│       ├── cli.py                    # Unified CLI: `htan <command>`
│       ├── config.py                 # Credential management (3-tier: env > keychain > file)
│       ├── query/
│       │   ├── portal.py             # Portal ClickHouse queries
│       │   └── bq.py                 # BigQuery queries
│       ├── download/
│       │   ├── synapse.py            # Synapse downloads
│       │   └── gen3.py               # Gen3/CRDC DRS downloads

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ncihtan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
