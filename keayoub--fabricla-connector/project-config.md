---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install in development mode
pip install -e ".[dev]"

# Run all tests
pytest

# Run a single test file
pytest tests/test_collectors.py

# Run a single test by name
pytest tests/test_collectors.py::test_function_name

# Run only unit tests (exclude integration/slow)
pytest -m "not integration and not slow"

# Run with coverage
pytest --cov=src

# Lint
flake8 src/

# Format
black src/

# Type check
mypy src/
```

Tests require a `tests/.env` file — copy `.env.example` to `tests/.env` and fill in values. By default, tests run with mocked APIs (`USE_MOCK_API=true`).

The CLI entry point is `fabric-monitor` (maps to `fabricla_connector.workflows:main`).

## Fabric-compatible runtime environments

The project must match the Microsoft Fabric runtime spec when running inside Fabric. Use the setup scripts to create a matching local venv:

```bash
# Interactive setup (downloads official Synapse Spark Runtime requirements)
cd setup && python setup_fabric_environment.py

# Windows shortcut
cd setup && setup_fabric_environment.bat
```

Two runtime versions are supported:
- **Runtime 1.2** — Python 3.10 + Spark 3.4 → creates `.fabric-env-1.2/`
- **Runtime 1.3** — Python 3.11 + Spark 3.5 → creates `.fabric-env-1.3/`

Activate on Windows: `.fabric-env-1.3\Scripts\activate.bat`

## Environment variables

All config comes from a single `.env.example` at the repo root. Copy it to wherever you need it:

```bash
cp .env.example .env            # root / local development
cp .env notebooks/.env          # for notebook runs
cp .env tests/.env              # for test runs
```

Key variable groups (see `.env.example` for the full list):

| Group | Variables |
|-------|-----------|
| Fabric auth | `FABRIC_TENANT_ID`, `FABRIC_APP_ID`, `FABRIC_APP_SECRET` |
| Fabric target | `FABRIC_WORKSPACE_ID`, `FABRIC_CAPACITY_ID` |
| Azure Monitor | `AZURE_MONITOR_DCE_ENDPOINT`, `AZURE_MONITOR_DCR_IMMUTABLE_ID` |
| Stream names | `AZURE_MONITOR_STREAM_LIVY_SESSION`, `AZURE_MONITOR_STREAM_SPARK_LOGS`, etc. |
| Collection | `LOOKBACK_HOURS` (default 24), `CHUNK_SIZE` (default 1000) |

When deploying with Terraform, get the DCE endpoint and DCR immutable ID from outputs:
```bash
cd infra/terraform && terraform output -json
```

The `DefaultAzureCredential` auth chain for ingestion: env vars (`AZURE_CLIENT_*`) → Managed Identity → VS Code → Azure CLI → PowerShell. For local testing, `az login` is the simplest path.

## Building and deploying the package to Fabric

```bash
# Build wheel only (validates package)
python tools/test_local_build.py --workspace-id <WS_ID> --environment-id <ENV_ID> --skip-upload

# Build and upload to Fabric staging
python tools/test_local_build.py --workspace-id <WS_ID> --environment-id <ENV_ID>

# Build, upload, and auto-publish (immediate activation)
python tools/test_local_build.py --workspace-id <WS_ID> --environment-id <ENV_ID> --publish

# Upload an existing wheel directly
python tools/upload_wheel_to_fabric.py --workspace-id <WS_ID> --environment-id <ENV_ID> --file dist/fabricla_connector-1.0.2-py3-none-any.whl --use-default-credential
```

Auth precedence for upload tools: `--token` (bearer) → service principal (`--client-id/--client-secret/--tenant-id`) → `--use-default-credential`.

## Fabric item creation tools

`tools/` also includes scripts to create Fabric items via REST API, all supporting the same three auth methods and a `--dry-run` flag:

```bash
# Create a data pipeline
python tools/create_fabric_pipeline.py --workspace-id <WS_ID> --pipeline-file tools/samples/sample_pipeline.json --use-default-credential

# Create a warehouse (sizes: small=DW100c, medium=DW200c, large=DW400c)
python tools/create_fabric_warehouse.py --workspace-id <WS_ID> --size medium --use-default-credential

# Create a Dataflow Gen2 (optionally with a Power Query M file)
python tools/create_fabric_dataflow_gen2.py --workspace-id <WS_ID> --mashup-file tools/samples/sample_mashup.pq --name "My ETL" --use-default-credential
```

## Troubleshooting

| Symptom | Likely cause |
|---------|-------------|
| "Missing required environment variables" | `.env` not loaded or wrong variable name |
| "Upload failed with 4xx" | Wrong DCR immutable ID, stream name mismatch, or missing "Monitoring Metrics Publisher" role |
| No data in Log Analytics | Wait 2–5 min; table name must include `_CL` suffix |
| `notebookutils` ImportError | Code is running locally, not in Fabric — expected, handled by `is_running_in_fabric()` |
| Auth failure on upload | Service principal needs `Fabric.ReadWrite.All` and workspace membership |

## Architecture

The package lives in `src/fabricla_connector/` and follows a strict layered pipeline:

```
Fabric REST APIs
     ↓
collectors/        — pull raw data from Fabric APIs
     ↓
mappers/           — transform raw API responses into Log Analytics schema
     ↓
ingestion/         — batch, chunk (1MB limit), retry, POST to Azure Monitor DCE
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Keayoub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
