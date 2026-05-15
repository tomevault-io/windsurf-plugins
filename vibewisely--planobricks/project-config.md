---
trigger: always_on
description: Planogram compliance dashboard for Retail/CPG, built on Databricks. Uses AI vision
---

# PlanoBricks

Planogram compliance dashboard for Retail/CPG, built on Databricks. Uses AI vision
(Foundation Model API + Claude Haiku 4.5) to detect shelf products and classify brands,
then verifies compliance against schematic planogram references using Needleman-Wunsch
sequence alignment. Multi-store capable with AI-powered schematic creation from images.

**Full product specification**: See `PRD.md` for the comprehensive Product Requirements Document.

## Product Domain

- **Planogram**: Expected shelf layout defining which brands go where, organized by shelf rows
- **Schematic**: Reference planogram built from multi-image consensus (auto) or user-created (custom)
- **Realogram**: Actual shelf layout captured via photograph, compared against the schematic
- **Compliance score**: Ratio of correctly placed products to expected (0.0–1.0) via NW alignment
- **Store**: A location grouping for shelf images and schematics (e.g., Store A, Store B)
- **Brand identification pipeline**: `ai_query('databricks-claude-haiku-4-5', ...)` on 13K+ product crops
- **Unity Catalog**: `serverless_stable_wunnava_catalog` catalog with `planobricks_reference` schema
- **SchematicKey**: Tuple of `(planogram_id, num_shelves, shelf_rank)` — e.g., `P01/3s/R1`

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Databricks App (Dash 4.0 + dash-bootstrap-components)      │
│  ┌──────────┐ ┌──────────────┐ ┌──────────┐ ┌───────────┐  │
│  │ Overview  │ │  Inspector   │ │  Editor  │ │  Dataset  │  │
│  │ (KPIs,   │ │  (Photo +    │ │(Schematic│ │  (Info,   │  │
│  │  charts, │ │  bounding    │ │ CRUD +   │ │  stats)   │  │
│  │  table)  │ │  boxes +     │ │ AI image │ │           │  │
│  │          │ │  schematic + │ │ upload)  │ │           │  │
│  │          │ │  crop/commit)│ │          │ │           │  │
│  └──────────┘ └──────────────┘ └──────────┘ └───────────┘  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Store Selector (navbar) — multi-store switching      │   │
│  └──────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│  grocery_data.py   — Data loading, caching, per-store data  │
│  planogram_engine.py — Row clustering, NW alignment, scores │
│  planogram_store.py — Schematic CRUD (local + UC Volume)    │
│  store_manager.py  — Multi-store management & persistence   │
├─────────────────────────────────────────────────────────────┤
│  UC Volumes (images, schematics JSON)                       │
│  Foundation Model API (brand detection from shelf images)    │
│  Databricks Asset Bundles (deployment)                      │
└─────────────────────────────────────────────────────────────┘
```

## Project Structure

```
planobricks/
├── src/app/                    # Dash application (deployed to Databricks Apps)
│   ├── app.py                  # Main Dash app — layout, all callbacks (~1800 lines)
│   ├── grocery_data.py         # Dataset parser, per-store caching, compliance aggregation
│   ├── planogram_engine.py     # NW alignment, schematic builder, compliance computation
│   ├── planogram_store.py      # Schematic JSON persistence (local + UC Volume, per-store)
│   ├── store_manager.py        # Multi-store CRUD and persistence
│   ├── app.yaml                # Databricks App runtime config (env vars)
│   ├── requirements.txt        # App dependencies (dash-bootstrap, plotly, databricks-sdk)
│   └── data/                   # Bundled annotation data
│       ├── annotation.txt      # 354 shelf images × product bounding boxes
│       ├── enriched_products.csv  # AI-identified brand per product (13K rows)
│       └── schematics.json     # Auto + custom schematic planograms
├── notebooks/
│   └── 01_brand_identification.py  # Brand classification via ai_query + Claude Haiku
├── resources/
│   └── planobricks_app.yml     # DABs app resource definition
├── scripts/                    # Test/verification scripts
├── tests/                      # Test suite
├── databricks.yml              # DABs bundle config (dev target, workspace profile)
├── pyproject.toml              # uv/hatch project config, ruff settings
├── PRD.md                      # Full Product Requirements Document
├── CLAUDE.md                   # This file — project context for AI assistants
└── README.md                   # User-facing documentation with screenshots
```

## Key Modules

### `app.py` — Dash Application
- **4 tabs**: Compliance Overview, Shelf Inspector, Schematic Editor, Dataset
- **Store selector** in navbar with create-store modal
- **Shelf Inspector**: actual photo + bounding box overlay + schematic reference + crop slider
  for re-evaluating compliance on a column sub-range (commit/reset workflow)
- **Schematic Editor**: CRUD (new, clone, delete, reset-to-auto), row editing via pipe-separated
  brand lists, AI image upload for automated brand detection
- **AI Brand Detection**: uploads shelf image → calls Foundation Model API
  (`databricks-claude-haiku-4-5`) via `w.api_client.do()` REST call → parses response into
  schematic rows → saves as custom schematic
- All tab content rendered in the initial layout (not dynamically) so Dash's client-side

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vibewisely/planobricks](https://github.com/vibewisely/planobricks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
