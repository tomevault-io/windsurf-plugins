---
trigger: always_on
description: The **Fabric Metadata-Driven Framework (FMD)** is an open-source Microsoft Fabric framework that automates, orchestrates, and standardises metadata-driven data pipelines. It targets a **Lakehouse-first / Medallion architecture** (Landing Zone → Bronze → Silver → Gold) and is fully deployed into Microsoft Fabric using two Jupyter deployment notebooks.
---

# Copilot Instructions — FMD Framework

## What this repository is

The **Fabric Metadata-Driven Framework (FMD)** is an open-source Microsoft Fabric framework that automates, orchestrates, and standardises metadata-driven data pipelines. It targets a **Lakehouse-first / Medallion architecture** (Landing Zone → Bronze → Silver → Gold) and is fully deployed into Microsoft Fabric using two Jupyter deployment notebooks.

Key resources:
- Full docs: <https://erwindekreuk.com/fmd-framework/>
- Wiki: <https://github.com/edkreuk/FMD_FRAMEWORK/wiki>
- Deployment guide: [`FMD_FRAMEWORK_DEPLOYMENT.md`](../FMD_FRAMEWORK_DEPLOYMENT.md)
- Business Domain guide: [`FMD_BUSINESS_DOMAIN_DEPLOYMENT.md`](../FMD_BUSINESS_DOMAIN_DEPLOYMENT.md)

---

## Repository layout

```
/
├── .github/                      # GitHub metadata (issue templates, this file)
├── Images/                       # Documentation images
├── Taskflow/                     # Fabric Taskflow JSON + assignment diagram
├── config/                       # Environment-specific config (item_config.yaml)
├── demodata/                     # Sample CSV files (e.g. customer.csv)
├── setup/                        # Deployment notebooks (run these in Fabric)
│   ├── NB_SETUP_FMD.ipynb           – deploys the FMD Framework workspaces & artifacts
│   └── NB_SETUP_BUSINESS_DOMAINS.ipynb – deploys Business Domain workspaces
└── src/                          # All Fabric artifacts (stored as source files)
    ├── Config_Database/          – SQL Database definition (schemas: integration, execution, logging)
    ├── ENV_FMD.Environment/      – Fabric Environment (Python libraries, Spark config)
    ├── NB_*.Notebook/            – PySpark notebooks (notebook-content.py + .platform)
    ├── PL_*.DataPipeline/        – Fabric Data Pipelines (JSON)
    ├── SQL_FMD_FRAMEWORK.SQLDatabase/ – SQL project (.sqlproj)
    ├── VAR_CONFIG_FMD.VariableLibrary/ – runtime config variables
    ├── VAR_FMD.VariableLibrary/  – default framework variables
    └── business_domain/          – Gold-layer notebooks + variable libraries
```

---

## Architecture overview

### Workspace separation
The framework uses **three separate Fabric workspaces**:

| Workspace | Naming convention | Purpose |
|---|---|---|
| Code | `<DOMAIN> CODE (D/P)` | Notebooks, pipelines, variable libraries |
| Data | `<DOMAIN> DATA (D/P)` | Lakehouses (LH_DATA_LANDINGZONE, LH_BRONZE_LAYER, LH_SILVER_LAYER) |
| Configuration | `FMD_FRAMEWORK_CONFIGURATION` | SQL_FMD_FRAMEWORK database, deployment notebooks |

Business Domains add two more: a Gold/Reporting workspace and a separate Business Domain workspace.

### Medallion layers

| Layer | Lakehouse | Description |
|---|---|---|
| Landing Zone | `LH_DATA_LANDINGZONE` | Raw files ingested from source systems |
| Bronze | `LH_BRONZE_LAYER` | Cleansed Delta tables with hash-based change tracking |
| Silver | `LH_SILVER_LAYER` | SCD Type 2 Delta tables (full history) |
| Gold | Domain-specific lakehouse | Aggregated / modelled data for reporting |

### Pipeline naming convention
| Prefix | Meaning |
|---|---|
| `PL_FMD_LDZ_COMMAND_*` | Commands a data copy from a specific source type |
| `PL_FMD_LDZ_COPY_FROM_*` | Actual copy pipeline for a source type |
| `PL_FMD_LOAD_*` | Orchestration pipeline (LANDINGZONE / BRONZE / SILVER / ALL) |

---

## Configuration database (`SQL_FMD_FRAMEWORK`)

All pipeline metadata is stored in the **Fabric SQL Database**. The three schemas are:

### `integration` schema — what to load
| Table | Key columns |
|---|---|
| `DataSource` | Source connection metadata |
| `Connection` | Fabric connection IDs |
| `Workspace` | Workspace GUIDs |
| `Lakehouse` | Lakehouse GUIDs per workspace/layer |
| `LandingzoneEntity` | Source entity config (SourceName, FilePath, FileType, IsIncremental, …) |
| `BronzeLayerEntity` | Bronze target config (PrimaryKeys, CleansingRules, …) |
| `SilverLayerEntity` | Silver target config (CleansingRules, …) |
| `Pipeline` | Pipeline metadata |

**Important stored procedure**: `integration.sp_UpsertLandingzoneBronzeSilver` registers a full LDZ → Bronze → Silver entity in a single transactional call.

### `execution` schema — what is queued/running
| Table | Purpose |
|---|---|
| `PipelineLandingzoneEntity` | Files detected in landing zone, pending processing |
| `PipelineBronzeLayerEntity` | Bronze entities queued for processing |
| `LandingzoneEntityLastLoadValue` | Watermark for incremental loads |

### `logging` schema — audit trail
| Table | Key columns |
|---|---|
| `PipelineExecution` | WorkspaceGuid, PipelineRunGuid, EntityId, EntityLayer, LogType, LogData |
| `NotebookExecution` | Notebook-level audit rows |
| `CopyActivityExecution` | Copy activity metrics |

---

## Notebooks

All notebooks live in `src/NB_*.Notebook/` and are saved as `notebook-content.py` (PySpark, kernel `synapse_pyspark`).

| Notebook | Role |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edkreuk/FMD_FRAMEWORK](https://github.com/edkreuk/FMD_FRAMEWORK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
