---
trigger: always_on
description: A **customer-deployable** Databricks App that assesses a workspace's maturity for
---

# Genie Ontology Readiness — Deployment Instructions

A **customer-deployable** Databricks App that assesses a workspace's maturity for
**Genie Ontology / Unity Catalog Business Semantics**, explains each capability
(technical + business value), recommends best practices, and generates tailored
enablement + adoption plans via the customer's **own Foundation Model API**.

It **reads the existing environment** (Unity Catalog `information_schema`, system
tables, Genie/Domains REST). It does **not** seed demo data. Full-stack app:
**FastAPI** backend + **React/Vite** frontend, deployed with **Databricks Asset
Bundles** + `post_deploy.py`.

> **Product accuracy:** Genie Ontology is the *learned* enterprise context layer
> (gated / Private Preview) built on top of the customer's *governed* UC Business
> Semantics (metric views, Pages, domains — largely GA). The foundation
> **feeds** the ontology. The app never positions the learned layer as GA.

## Prerequisites

- **Databricks workspace** with Unity Catalog (a serverless workspace is recommended).
- **Databricks CLI 0.287+** — `databricks --version`.
- **Authenticated profile**: `databricks auth login --host <workspace-url> --profile <name>`.
- **Node.js 18+** and **Python 3.11** with `pip install "databricks-sdk>=0.36.0"`.
- A **SQL Warehouse** (its `id` is needed for assessment queries).

## Deployment (3 commands)

All commands run from the repository root.

### 1. Build the frontend

```bash
cd app/frontend && npm install && npm run build && cd ../..
```

### 2. Deploy infrastructure (app + SQL warehouse resource)

```bash
databricks bundle deploy -t dev \
  --profile <your-profile> \
  --var="warehouse_id=<your_warehouse_id>"
```

Each environment is a **bundle target** with its own Terraform state and a
**fixed app name** (`dev` → `genie-ontology-readiness-dev`, `stg` → `…-stg`,
`prod` → the bare `genie-ontology-readiness`). Pick the environment with
`-t <target>`; there is **no `--var app_name`** — the name is pinned per target
in `databricks.yml` on purpose, so a deploy can't rename/destroy another env's
app. For a clean production name, deploy `-t prod`. Routine redeploys of an
existing app should use the non-destructive `databricks apps deploy <name>`;
reserve `bundle deploy` for first-time create / infra changes.

### 3. Post-deploy (render app.yml, grant SP read access, publish)

```bash
export DATABRICKS_PROFILE=<your-profile>
export TARGET=dev   # dev (default) | stg | prod — must match the `-t` above;
                    # the app name is derived from the target, don't set it here
export WAREHOUSE_ID=<your_warehouse_id>
# Per-user history (assessment runs + saved plans) — recommended:
export USE_LAKEBASE=true
export LAKEBASE_INSTANCE_NAME=<your-instance>   # an existing Lakebase instance to reuse (leave unset to skip history)
export LAKEBASE_DATABASE=ontology_readiness     # a dedicated DB created on that instance
# Optional:
# export ASSESS_CATALOGS="sales,finance"     # restrict scope (default: all non-system catalogs)
# export GENIE_SPACE_ID=<id>                  # enable the live Genie answer test (pillar 5)
# export BRAND_NAME="Acme"                    # header branding
# export FORCE_SP=true                        # SP-only mode: never attempt OBO, run every read as the app SP

python3 scripts/post_deploy.py   # requires: pip install "databricks-sdk>=0.36.0" asyncpg
```

`post_deploy.py` renders `app/app.yml`, grants the **app service principal** read
access (system schemas + assessed catalogs), and republishes the app. When
`USE_LAKEBASE=true` it also creates the app's Postgres database on the shared
Lakebase instance, attaches the `lakebase-db` app resource, and pins the
connection env. Assessment history and generated plans are then persisted
**per user** (keyed by the `X-Forwarded-Email` identity Databricks Apps inject)
and survive across sessions. Scores are a deterministic function of the technical
findings, so repeated runs on the same workspace are directly comparable **for a
given identity**. Under OBO the findings reflect the viewing user's grants, so
signals gated behind grants a viewer lacks (notably the workspace-wide Adoption
system tables) can read lower for a restricted viewer than for the SP — use
`FORCE_SP=true` when you need a single, viewer-independent workspace-wide score.

## Auth: on-behalf-of-user by default, service-principal fallback

The assessment runs **on-behalf-of-user (OBO)** by default, falling back to the
app **service principal (SP)** whenever OBO isn't available or a specific read
can't succeed as the viewer.

**On-behalf-of-user (OBO):** if the workspace admin enables **user authorization**
(Public Preview) and the app carries the `sql` user scope, Databricks forwards the
viewer's token in `x-forwarded-access-token`. The app then runs **every** signal —
both `information_schema`/tags **and** the system-table signals
(`system.access.audit`, `system.query.history`, `system.access.table_lineage`) — as
the **logged-in user**, so the assessment reflects what that user can see. The
policy lives in `sql_client.execute_sql`:

1. `force_sp=True` → **SP-only override** (no OBO attempt); used for reads the OBO
   token can't do — the Genie REST API (not covered by the `sql` scope) and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [databricks-solutions/genie-ontology-readiness](https://github.com/databricks-solutions/genie-ontology-readiness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
