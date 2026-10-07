---
trigger: always_on
description: **Danish Democracy Data (dbt + DuckDB Demo)** — a data engineering pipeline that
---

# CLAUDE.md — Project Context for AI Assistants

## Project Overview

**Danish Democracy Data (dbt + DuckDB Demo)** — a data engineering pipeline that
ingests open data from the **Danish Parliament OData API** (18 entities) and the
**Rfam public MySQL database** (7 tables), transforms it through a
**Bronze → Silver → Gold medallion architecture** inside DuckDB, and optionally
exports the result as Delta Lake tables to Microsoft Fabric OneLake.

- **Repo:** `edwinweber/dbt_duckdb_demo`
- **Default branch:** `main`
- **License:** see LICENSE file
- A learning/reference project demonstrating common data engineering patterns on a low-cost, open-source stack.

## Tech Stack

| Layer           | Technology                                           |
|-----------------|------------------------------------------------------|
| Language        | Python ≥3.12                                         |
| Orchestration   | Dagster ≥1.12 (software-defined assets)              |
| Extraction      | dlt ≥1.24 (Data Load Tool)                           |
| Transformation  | dbt-core ≥1.10,<1.12 + dbt-duckdb ≥1.10             |
| Query engine    | DuckDB ≥1.5.1,<1.6                                   |
| Silver storage  | DuckDB native tables (default) **or** DuckLake (Parquet + catalog) |
| Data quality    | dbt-utils 1.3.0 + dbt built-in tests                 |
| Cloud storage   | Microsoft Fabric OneLake (ADLS Gen2 / Delta Lake)    |
| Export          | DuckDB `delta_scan` (dedup read) + deltalake ≥1.5 / PyArrow ≥17 (write) |
| SQL source      | SQLAlchemy ≥2.0, PyMySQL ≥1.1                        |
| Container       | Docker + Docker Compose                               |
| Testing         | pytest ≥8.0                                          |

## Architecture

```text
Extraction (dlt)
   ↓  JSON / Parquet files
Bronze (DuckDB views over raw files)
   ↓  hash-based CDC, SCD Type 2
Silver (DuckDB native tables OR DuckLake Parquet + _cv current-version views)
   ↓  star-schema modeling
Gold (DuckDB views — facts & dimensions)
   ↓  Delta Lake export
OneLake / local filesystem
```

The **Silver storage format** is switchable via `SILVER_STORAGE_FORMAT`
(`duckdb` default | `ducklake`) — see *Silver Storage Format* below. This is
orthogonal to `STORAGE_TARGET`, which only controls the Delta Lake **export**
destination (local vs OneLake).

Orchestrated by **Dagster** (two daily schedules: 06:00 full pipeline +
08:00 data engineering, Europe/Copenhagen, disabled by default). DuckLake
catalog cleanup is a separate **manual** job (no schedule).

## Directory Structure

```text
├── ddd_python/                  Python package (the code)
│   ├── ddd_dagster/             Dagster assets, jobs, schedules, sensors, resources
│   ├── ddd_dlt/                 dlt extraction pipelines + Delta Lake export
│   ├── ddd_dbt/                 dbt model generator + dbt runner + DuckDB init
│   └── ddd_utils/               Configuration, env vars, Azure/Fabric clients
├── dbt/                         dbt project
│   ├── models/bronze/           53 views (read_json_auto over raw files)
│   ├── models/silver/           50 models (CDC tables + _cv views)
│   ├── models/gold/             19 models (star-schema views)
│   ├── models/data_engineering/ 8 observability models (Dagster SQLite)
│   ├── macros/                  9 Jinja macros (model factories, hash, CDC)
│   ├── seeds/                   Seed CSVs (Danish public holidays, source registry)
│   └── dbt_project.yml          Project config + variables
├── tests/                       pytest tests (15 modules, 132 tests)
├── duckdb/                      DuckDB init scripts (extensions + Azure secret)
├── dlt/pipelines_dir/           dlt incremental state (git-ignored)
├── data/                        Local storage root (git-ignored)
│   └── Files/{Bronze,Silver,Gold}/
├── documentation/               Handbook markdown + build scripts
├── docker-compose.yml           Services: 'run' (one-off) + 'dagster' (UI)
├── Dockerfile                   Python 3.12 + DuckDB CLI v1.5.3
├── pyproject.toml               Dependencies + build config
├── workspace.yaml               Dagster workspace (loads ddd_dagster.definitions)
└── .env.example                 Template for environment variables
```

## Data Sources

### Danish Parliament API (DDD)

- **Base URL:** `https://oda.ft.dk/api`
- **18 OData entities:** Afstemning, Afstemningstype, Aktør, Aktørtype, Møde,
  Mødestatus, Mødetype, Periode, Sag, Sagstrin, SagstrinAktør,
  Sagstrinsstatus, Sagstrinstype, Sagskategori, Sagsstatus, Sagstype,
  Stemme, Stemmetype
- **Incremental (6):** Aktør, Møde, Sag, Sagstrin, SagstrinAktør, Stemme
  — filtered by `$filter=opdateringsdato ge DateTime'...'`
- **Full-extract (12):** the remaining small lookup tables
- **All primary keys:** `id`

### Rfam MySQL Database

- **Connection:** `mysql+pymysql://rfamro@mysql-rfam-public.ebi.ac.uk:4497/Rfam`
  (public read-only)
- **7 tables:** family, genome, clan, clan_membership, author,
  literature_reference, dead_family
- **Incremental (2):** family (pk `rfam_acc`, date `updated`), genome
  (pk `upid`, date `updated`)
- **Full-extract (5):** remaining tables (no date column)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edwinweber/dbt_duckdb_demo_public](https://github.com/edwinweber/dbt_duckdb_demo_public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
