---
trigger: always_on
description: <!-- Copilot instructions for the Tableau to Power BI migration project -->
---

<!-- Copilot instructions for the Tableau to Power BI migration project -->

# Project: Tableau to Power BI Migration

Automated migration of Tableau workbooks (.twb/.twbx) to Power BI projects (.pbip) in PBIR v4.0 format with TMDL semantic model.

## Architecture — 2-Step Pipeline

```
.twbx --> [Extraction] --> 23 JSON files --> [Generation] --> .pbip (PBIR + TMDL)
```

1. **Extraction** (`tableau_export/`): Parses Tableau XML, extracts worksheets/dashboards/datasources/calculations/parameters/filters/stories/actions/sets/groups/bins/hierarchies/sort_orders/aliases/custom_sql/user_filters/hyper_files/datasource_filters/custom_geocoding/published_datasources/data_blending/table_extensions/linguistic_schema
2. **Generation** (`powerbi_import/`): Produces the complete .pbip project (BIM → TMDL, PBIR v4.0 report, Power Query M, visuals, filters, bookmarks)

## Project Structure

- **tableau_export/**: Tableau XML extraction and parsing + DAX formula conversion
  - `extract_tableau_data.py`: Main orchestrator, parses TWB/TWBX, extracts 23 object types
  - `datasource_extractor.py`: Datasource extraction (connections, tables, columns, calculations, relationships)
  - `dax_converter.py`: 133 Tableau → DAX formula conversions (89 regex patterns + 44 converter functions: LOD, table calcs, security, etc.)
  - `m_query_builder.py`: Power Query M generator (49 connector types + 43 transformation generators: rename, filter, aggregate, pivot/unpivot, join, union, sort, conditional columns — chainable via `inject_m_steps()`)
  - `prep_flow_parser.py`: Tableau Prep flow parser (.tfl/.tflx → Power Query M) — DAG traversal, Clean/Join/Aggregate/Union/Pivot steps, expression converter, merge with TWB datasources
  - `prep_flow_analyzer.py`: Per-flow metadata extraction for lineage analysis — parses individual .tfl/.tflx files and extracts FlowProfile objects (inputs, outputs, transforms with action-level detail, DAG statistics, complexity signals). Extracts 18 operation types from Clean steps (rename, filter, remove columns, calculated fields, etc.), join key columns, aggregate group-by/agg columns, script types, output columns. `analyze_flow()`, `analyze_flows_bulk()`
  - `server_client.py`: Tableau Server/Cloud REST API client — PAT/password auth, workbook download, datasource listing, batch download, regex search, context manager, paginated API fetching (`_paginated_get`), 9 new endpoints: `list_users`, `list_groups`, `list_views`, `get_workbook_connections`, `list_schedules`, `get_site_info`, `list_prep_flows`, `download_prep_flow`, `get_server_summary`, `get_workbook_extract_tasks`, `get_workbook_subscriptions`
  - `hyper_reader.py`: Hyper file data loader — 3-tier reader chain (tableauhyperapi → sqlite3 → binary header scan), schema discovery, type mapping (28 types), M expression generation (#table inline / Csv.Document), CSV export (`export_hyper_to_csv`), relationship inference (`infer_hyper_relationships`), metadata enrichment with recommendations
  - `pulse_extractor.py`: Tableau Pulse metric extractor — parses Pulse metric definitions from TWB XML (metric name, measure, time dimension, filters, goals)
  - `safe_xml.py`: Safe XML parsing utilities — XXE-protected ElementTree parser for secure TWB/TWBX processing
- **powerbi_import/**: Power BI project generation
  - `pbip_generator.py`: .pbip generator (PBIR v4.0, visuals, filters, bookmarks, slicers, textbox, image, pages shelf, number format conversion, drill-through pages)
  - `tmdl_generator.py`: Unified semantic model generator — direct Tableau → TMDL (tables, columns, measures, relationships, hierarchies, sets/groups/bins, parameters, RLS, dataCategory, isHidden, calculation groups, field parameters, M-based calculated columns)
  - `visual_generator.py`: Visual container generator — 190 visual type mappings (136 VISUAL_TYPE_MAP + 16 APPROXIMATION_MAP + 38 CUSTOM_VISUAL_GUIDS), PBIR-native config templates, data role definitions, query state builder, slicer sync groups, cross-filtering disable, action button navigation, TopN filters, sort state, reference lines, conditional formatting
  - `import_to_powerbi.py`: Generation pipeline orchestrator (supports `--output-dir`, `--output-format fabric` for shared models)
  - `m_query_generator.py`: Sample data M query generator
  - `assessment.py`: Pre-migration readiness assessment — 9 categories (datasource, calculation, visual, filter, data model, interactivity, extract, scope, connection string audit), pass/warn/fail scoring
  - `server_assessment.py`: Server-level portfolio assessment — per-workbook GREEN/YELLOW/RED classification, 8-axis complexity computation, effort estimation, migration wave planning, connector census, HTML executive dashboard
  - `strategy_advisor.py`: Migration strategy advisor — recommends Import/DirectQuery/Composite based on 7 signals
  - `validator.py`: Artifact validator — validates .pbip projects (JSON, TMDL, report structure) before opening in PBI Desktop
  - `migration_report.py`: Per-item fidelity tracking and migration status reporting
  - `goals_generator.py`: PBI Goals/Scorecard generator — converts Tableau Pulse metrics to Power BI Goals JSON (goal name, current value measure, target, status rules, sparkline)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyphou/Tableau-To-PowerBI](https://github.com/cyphou/Tableau-To-PowerBI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
