---
trigger: always_on
description: This file helps AI coding agents (Cursor, Copilot, etc.)
---

# Agent Guide for SoloLakehouse

This file helps AI coding agents (Cursor, Copilot, etc.)
understand the project quickly. Read this before making any changes.

## What This Project Is

SoloLakehouse is a **reference implementation** of a Lakehouse architecture,
not a framework or library. It demonstrates how platforms like Databricks and
Snowflake work internally, using only open-source tools on a single Docker
Compose node.

**Current: v2.5 single-track baseline** — orchestrated platform with Dagster assets/schedules/UI, Iceberg Gold in Trino, and mandatory OpenMetadata + Superset in the default stack (see `docs/roadmap.md`).  
**Next target (v3.0):** production infrastructure and governance hardening (multi-environment deployment, secrets/access governance, SLO/alerting, release promotion controls).

**Domain:** Financial data engineering + ML (ECB interest rates + DAX stock index).

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Object Storage | MinIO (S3-compatible) | RELEASE.2025-09-07T16-13-09Z |
| Metadata DB | PostgreSQL | 17 |
| Table Catalog | Apache Hive Metastore (standalone) | 4.0.0 |
| Query Engine | Trino (Hive + Iceberg catalogs) | 480 |
| Table format (Gold) | Apache Iceberg (via Trino) | — |
| Data catalog | OpenMetadata | 1.5.x |
| BI / SQL UI | Apache Superset | 6.0.0 |
| ML Tracking | MLflow | 3.10.1 |
| Orchestration | Dagster | 1.7.x (Python < 3.13) / 1.12.x (Python ≥ 3.13) |
| Language | Python | 3.13+ |
| Validation | Pydantic v2 | 2.12.5 |
| Data Format | Parquet (snappy) via PyArrow; Gold also exposed as Iceberg | 23.0.1 |
| Logging | structlog | 25.5.0 |
| Testing | pytest | 9.0.2 |

## Commands

```bash
make up          # Start all Docker services + init MinIO buckets (includes Dagster services)
make down        # Stop services (data preserved under docker/data/)
make pipeline    # Run Dagster full_pipeline_job (v2.5 default path)
make dagster-ui  # Open Dagster UI (http://localhost:3000)
make verify      # Health-check all services
make test        # Run unit tests (pytest, no Docker needed)
make lint        # ruff (CI)
make typecheck   # mypy on ingestion/, transformations/, ml/, scripts/, dagster/ (install requirements-dagster.txt so the local dagster/ folder does not shadow PyPI dagster)
make clean       # Stop services + delete docker/data/ + purge legacy named Docker volumes
```

## Project Layout

```
ingestion/
  collectors/         # One class per data source (ECBCollector, DAXCollector)
  schema/             # Pydantic v2 models for record validation
  quality/            # Bronze-layer quality check functions
  bronze_writer.py    # Writes validated data to MinIO as Parquet
  trino_sql.py          # Trino REST: Hive staging + Iceberg Gold refresh

transformations/
  ecb_bronze_to_silver.py   # ECB: type cleanup, forward-fill, rate_change_bps
  dax_bronze_to_silver.py   # DAX: weekend filter, daily_return
  silver_to_gold_features.py # Join ECB+DAX, build event-study features

ml/
  train_ecb_dax_model.py    # XGBoost/LightGBM with TimeSeriesSplit CV
  evaluate.py               # MLflow experiment runner (multiple hyperparams)

scripts/
  verify-setup.py           # Service health checks
  bootstrap-postgres.py     # Ensure DBs exist; TCP password check + align vs .env after docker-exec bootstrap
  prepare-docker-data-dirs.sh   # mkdir + perms for docker/data bind mounts
  purge-legacy-docker-volumes.sh # Remove pre-bind-mount Docker named volumes (after down)
  init-minio.sh             # Legacy bucket init (now handled by minio-init container)
  trino-entrypoint.sh       # Expands all Trino catalog *.properties templates

config/
  trino/catalog/hive.properties   # Template — uses ${S3_ACCESS_KEY}/${S3_SECRET_KEY}
  trino/catalog/iceberg.properties # Iceberg connector + Hive Metastore catalog
  trino/config.properties        # Trino coordinator settings
  postgres/init.sql              # Creates hive_metastore + mlflow databases

docker/
  docker-compose.yml        # Core platform services
  docker-compose.openmetadata.yml # OpenMetadata stack (included by default in Makefile)
  docker-compose.superset.yml # Superset stack (included by default in Makefile)
  data/                     # Bind-mounted runtime state (MinIO, Postgres, Dagster, OM; contents gitignored)
  dagster/                  # Dagster image build context
  hive-metastore/           # Custom Dockerfile + entrypoint (envsubst)
  mlflow/                   # Custom Dockerfile
  superset/                 # Custom Superset image + bootstrap config

dagster/
  assets.py                 # Software-defined assets, sensor, asset checks
  resources.py              # MinIO/config resources
  definitions.py            # Jobs/schedule/definitions registry
  workspace.yaml            # Dagster code location workspace
  dagster.yaml              # Dagster instance config (PostgreSQL storage)

tests/                      # Unit tests (mocked I/O, no Docker needed)
docs/                       # See docs/README.md — architecture, ADRs, roadmap, deployment
data/sample/                # Committed sample CSV for DAX
```

## Architecture Patterns — Follow These When Adding Code

### Collector Pattern (ingestion/collectors/)

```python
class NewCollector:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jiahong-Que-9527/SoloLakehouse](https://github.com/Jiahong-Que-9527/SoloLakehouse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
