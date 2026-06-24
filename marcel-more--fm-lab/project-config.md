---
trigger: always_on
description: You are an expert for FileMaker solutions. You help with the analysis and development of FileMaker applications. To do so, you have access to the metadata and descriptions of all objects in the solution the developer is working on.
---

# FileMaker XML Analysis

## Role

You are an expert for FileMaker solutions. You help with the analysis and development of FileMaker applications. To do so, you have access to the metadata and descriptions of all objects in the solution the developer is working on.

## Context

FileMaker is an integrated development tool that combines a database, user interface and script engine. The function `SaveCopyAsXML` exports the entire application structure without the contained user data. One XML file is produced per FileMaker file, containing an object catalog covering every component of the application.

To optimize access to the various subsections of the object catalog, we first convert the XML into a DuckDB database in which each object type is exposed as its own table. This enables fast queries across all objects and their relationships, since we can use SQL queries to quickly and flexibly access exactly the information we need.


## XML structure

**Supported XML versions:** SaXML v2.1.0.0+ (FileMaker 19+) with the root element `<FMSaveAsXML>`. The older SaXML v2.0.0.0 format (FileMaker 18.x) with the root element `<FMDynamicTemplate>` is not supported and is automatically skipped with a warning.

A description of the XML structure of the FileMaker file can be found in `docs/agents/xml-schema.md`.
This is the starting point of our conversion. Afterwards, all data from the XML that is relevant for our analysis lives in the DuckDB tables. For every analysis step we exclusively use the data from the DuckDB database.



## Building the database

The conversion runs as a **six-phase pipeline** (`sql/convert-xml/convert_xml_0N_<phase>.sql`),
orchestrated by the `convert-xml` skill. Only **Phase 1 reads the XML** — all later
phases work purely on the DuckDB tables (no `read_xml`), which keeps the parse load
and memory peak low.

| Phase | File | Reads | Produces |
|---|---|---|---|
| **P1 Extract** | `sql/convert-xml/convert_xml_01_extract.sql` | XML (`read_xml`) | Raw catalogs + raw-XML columns (`Step_XML`, `Object_XML`, `Parameters_XML`, …) |
| **P2 Resolve** | `sql/convert-xml/convert_xml_02_resolve.sql` | tables only | Reference tables (XMLStep/Layout/Calc-Refs, MBS/GetSub, PluginUsages) |
| **P3 Details** | `sql/convert-xml/convert_xml_03_details.sql` | tables only | Variable analysis (VariableUsages, VariablesCatalog) |
| **P4 Catalog** | `sql/convert-xml/convert_xml_04_catalog.sql` | tables only | ObjectCatalog + ObjectLinks |
| **P5 Homes** | `sql/convert-xml/convert_xml_05_homes.sql` | tables only | Cross-file resolution (ObjectHomes, TableOccurrenceResolution) + graph views (`LogicalLinks`, `ClusterEdges`) |
| **P6 Validate** | `sql/convert-xml/convert_xml_06_validate.sql` | tables only | Plausibility/consistency check views (`v_check_*`), queried by the post-processor |

P1 runs once per file; P2–P6 run once after all files are imported (batch-wide).

P5 also creates **graph views** (read-only helpers over the universal catalogs):
`LogicalLinks` (operational links, sub-objects hoisted to their container, containment
scaffold + orphans removed, **local variables `$x` excluded**) and the cluster-edge chain
`ClusterEdgesBase` (= `LogicalLinks` minus `BuiltinFunction`) → `ClusterGodNodes` (cross-cutting
"god-nodes": neighbours span ≥8 files **and** ≤40 % in their own file — generic MBS-plugin utilities
+ global config/auth fields; **Stufe D**) → `ClusterEdges` (= `ClusterEdgesBase` minus
`ClusterGodNodes`). `ClusterEdges` is the single source of truth for the community-detection edge
export (`tools/graph-export/graph_export_logical.sql`) and the `fm-graph-cluster` skill's
logical-degree/hub analysis. The god-node filter sits **only** in `ClusterEdges` (clustering), not in
`LogicalLinks` (the Explorer/where-used still shows god-nodes). `LogicalLinks` canonical definition
mirrored in [rest-api/templates/sql/graph_logical_links.sql](rest-api/templates/sql/graph_logical_links.sql).
**Local-variable filter (Stufe C):**
local variables are keyed per-script (`Scope_Anchor`=script) → degree-1 pendants that only clutter
the graph (≈34 % of cluster nodes) without bridging modules, so they are dropped from `LogicalLinks`.
Global (`$$`) / superglobal (`$$$`) variables stay (real cross-script bridges). The semantic
variable signal is unaffected — `fm-analyze`/`fm-graph-cluster` read variable names from
`VariableUsages`/`VariablesCatalog` (per script), not from the graph.

**`--split` (large files):** `convert-xml --batch --split` chunks each file's Phase 1
at top-level branch boundaries (the heavy `StepsForScripts` and `DDR_INFO` branches
are split into their own chunks) to lower the peak DOM memory. P2–P6 are unaffected
(table-only, batch-wide). The result is bit-identical to the unsplit run.

The database is stored in the `db/` directory. The file name is `fm_catalog.duckdb`.

**Recommended approach:** Use the `convert-xml` skill (runs the full pipeline):
- **Single file**: `convert-xml "MyDatabase.xml"`
- **All files (batch)**: `convert-xml --batch`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcel-more/fm-lab](https://github.com/marcel-more/fm-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
