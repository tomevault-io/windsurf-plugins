---
trigger: always_on
description: **ggsql** is a SQL extension for declarative data visualization based on Grammar of Graphics principles. It allows users to combine SQL data queries with visualization specifications in a single, composable syntax.
---

# ggsql System Architecture & Implementation Summary

## Overview

**ggsql** is a SQL extension for declarative data visualization based on Grammar of Graphics principles. It allows users to combine SQL data queries with visualization specifications in a single, composable syntax.

**Core Innovation**: ggsql extends standard SQL with a `VISUALISE` clause that separates data retrieval (SQL) from visual encoding (Grammar of Graphics), enabling terminal visualization operations that produce charts instead of relational data.

```sql
SELECT date, revenue, region FROM sales WHERE year = 2024
VISUALISE date AS x, revenue AS y, region AS color
DRAW line
SCALE x VIA date
SCALE y FROM (0, 100000)
LABEL title => 'Sales by Region', x => 'Date', y => 'Revenue'
```

**Statistics**:

- ~7,500 lines of Rust code (including PROJECT implementation)
- 507-line Tree-sitter grammar (simplified, no external scanner)
- Full bindings: Rust, C, Python, Node.js with tree-sitter integration
- Syntax highlighting support via Tree-sitter queries
- 916 total tests (174 parser tests, comprehensive builder and integration tests)
- End-to-end working pipeline: SQL → Data → Visualization
- Projectinate transformations: Cartesian, Flip, Polar
- VISUALISE FROM shorthand syntax with automatic SELECT injection

---

## Global Mapping Feature

ggsql supports global aesthetic mappings at the VISUALISE level that apply to all layers:

### Explicit Global Mapping

Map columns to specific aesthetics at the VISUALISE level:

```sql
SELECT * FROM sales WHERE year = 2024
VISUALISE date AS x, revenue AS y, region AS color
DRAW line
DRAW point
-- Both layers inherit x, y, and color mappings
```

### Implicit Global Mapping

Use column names directly when they match aesthetic names:

```sql
SELECT x, y FROM data
VISUALISE x, y
DRAW point
-- Equivalent to: VISUALISE x AS x, y AS y
```

### Wildcard Mapping

Map all columns automatically (resolved at execution time):

```sql
SELECT * FROM data
VISUALISE *
DRAW point
-- All columns mapped to aesthetics with matching names
```

### VISUALISE FROM Shorthand

Direct visualization from tables/CTEs (auto-injects `SELECT * FROM`):

```sql
-- Direct table visualization
VISUALISE FROM sales
DRAW bar
  MAPPING region AS x, total AS y

-- CTE visualization
WITH monthly_totals AS (
  SELECT DATE_TRUNC('month', sale_date) as month, SUM(revenue) as total
  FROM sales
  GROUP BY month
)
VISUALISE FROM monthly_totals
DRAW line
  MAPPING month AS x, total AS y
```

---

## System Architecture

### High-Level Flow

```
┌─────────────────────────────────────────────────────────────┐
│                       ggsql Query                            │
│  "SELECT ... FROM ... WHERE ... VISUALISE x, y DRAW ..."     │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
         ┌───────────────────────────────┐
         │        SourceTree             │
         │  (Parse once, reuse CST)      │
         │  • extract_sql()              │
         │  • extract_visualise()        │
         └───────────┬───────────────────┘
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
  ┌─────────────┐        ┌──────────────┐
  │  SQL Part   │        │   VIZ Part   │
  │ "SELECT..." │        │ "VISUALISE..." │
  └──────┬──────┘        └──────┬───────┘
         │                      │
         ▼                      ▼
  ┌─────────────┐        ┌──────────────┐
  │   Reader    │        │    Parser    │
  │  (DuckDB,   │        │ (tree-sitter)│
  │   Postgres) │        │   → AST      │
  └──────┬──────┘        └──────┬───────┘
         │                      │
         ▼                      │
  ┌─────────────┐               │
  │  DataFrame  │               │
  │  (Polars)   │               │
  └──────┬──────┘               │
         │                      │
         └──────────┬───────────┘
                    ▼
         ┌─────────────────────┐
         │      Writer          │
         │  (Vega-Lite, ggplot) │
         └──────────┬───────────┘
                    ▼
         ┌─────────────────────┐
         │   Visualization      │
         │  (JSON, PNG, HTML)   │
         └─────────────────────┘
```

### Key Design Principles

1. **Separation of Concerns**: SQL handles data retrieval, VISUALISE handles visual encoding
2. **Pluggable Architecture**: Readers and Writers are trait-based, enabling multiple backends
3. **Grammar of Graphics**: Composable layers, explicit aesthetic mappings, scale transformations
4. **Terminal Operation**: Produces visualizations, not relational data (like SQL's `SELECT`)
5. **Type Safety**: Strong typing through AST with Rust's type system

---

## Public API

### Quick Start

```rust
use ggsql::reader::{DuckDBReader, Reader};
use ggsql::writer::VegaLiteWriter;

// Create a reader
let reader = DuckDBReader::from_connection_string("duckdb://memory")?;

// Execute the ggsql query
let spec = reader.execute(
    "SELECT x, y FROM data VISUALISE x, y DRAW point"
)?;

// Render to Vega-Lite JSON
let writer = VegaLiteWriter::new();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [posit-dev/ggsql](https://github.com/posit-dev/ggsql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
