---
trigger: always_on
description: DuckDB v1.5 spatial/GIS analytics with 155+ ST_* functions. Use this skill for DuckDB, spatial queries, GeoParquet, geometry data, or geospatial analytics. Workflow: Discovery → Understanding → Analysis (Phase 1: glob/DESCRIBE/ST_Read_Meta, Phase 2: SUMMARIZE/parquet_metadata/CRS detection, Phase 3: targeted queries). Query profiling with EXPLAIN ANALYZE for troubleshooting. Spatial: CRS/EPSG transforms, automatic R-Tree indexes, distance/area calculations, spatial joins, H3/A5/S2 indexing, QUAD
---


# DuckDB v1.5 Skill

> DuckDB v1.5.0 "Variegata" — Released 2026-03-09.

## How to Think: Discovery → Understanding → Analysis

The most common mistake is jumping straight to queries. Instead, follow this workflow — it prevents wrong assumptions, bad joins, and wasted work.

### Phase 1: Discover the Data

Before writing any analytical query, find out what you're working with.

**Local files — use glob patterns to find them:**

```sql
-- Find what files exist (Parquet, CSV, JSON, GeoJSON, etc.)
FROM glob('data/*.parquet');
FROM glob('**/*.csv');
FROM glob('/path/to/project/**/*.geojson');
```

**Remote files — always load httpfs first, then probe:**

```sql
INSTALL httpfs; LOAD httpfs;
-- For S3, also set the region:
SET s3_region = 'us-west-2';

-- Remote Parquet (only reads metadata, not full file)
DESCRIBE FROM 'https://example.com/data.parquet';

-- S3 / Azure / GCS
DESCRIBE FROM 's3://bucket/path/file.parquet';

-- S3 buckets with dots in the name (e.g., source.coop) need path-style URLs,
-- because virtual-hosted style breaks SSL certificate validation:
SET s3_url_style = 'path';
DESCRIBE FROM 's3://us-west-2.opendata.source.coop/repo/data.parquet';

-- Hive-partitioned datasets — check partition structure
FROM parquet_metadata('s3://bucket/dataset/**/*.parquet') LIMIT 5;
```

**Databases — inspect what's available:**

```sql
-- Attached databases
SHOW ALL TABLES;
DESCRIBE table_name;

-- External DuckDB files (new in v1.5 — no ATTACH needed)
FROM read_duckdb('other.duckdb', table_name := 'my_table');

-- External databases via ODBC (new in v1.5, requires INSTALL odbc_scanner)
-- odbc_connect(conn_str) + odbc_query(conn, sql)
```

**Discover functions from DuckDB itself** — when unsure about a function signature:

```sql
-- Search for functions by name pattern
SELECT function_name, parameters, return_type
FROM duckdb_functions() WHERE function_name LIKE 'ST_Coverage%';

-- List all extensions and their status
FROM duckdb_extensions();

-- List all settings
FROM duckdb_settings() WHERE name LIKE '%geometry%';
```

**Geospatial files — check layers and metadata:**

```sql
LOAD spatial;
SELECT * FROM ST_Read_Meta('data.gpkg');       -- layers, CRS, field schemas
SELECT * FROM ST_Read_Meta('data.gdb');        -- ESRI File Geodatabase
```

### Phase 2: Understand the Shape

Once you know WHAT files exist, understand their structure before querying.

```sql
-- Schema: column names and types (instant, reads only metadata)
DESCRIBE FROM 'data.parquet';
DESCRIBE FROM 'data.csv';

-- Stats: min, max, approx_unique, avg, std, quartiles, nulls (scans full data)
SUMMARIZE FROM 'data.parquet';
SUMMARIZE FROM 'data.csv';

-- Quick sample: see actual values (avoids scanning everything)
FROM 'data.parquet' LIMIT 10;
FROM 'data.csv' LIMIT 10;

-- Row count estimate (fast for Parquet — reads metadata only)
SELECT count(*) FROM 'data.parquet';

-- For Parquet: row group structure, column statistics, file-level metadata
FROM parquet_metadata('data.parquet');
FROM parquet_schema('data.parquet');
FROM parquet_kv_metadata('data.parquet');  -- GeoParquet metadata lives here
```

This phase is critical because:
- DESCRIBE tells you column types — you'll know if geometry is WKB blob vs native GEOMETRY
- SUMMARIZE reveals data quality issues (null percentages, unexpected ranges, cardinality)
- A quick LIMIT sample shows actual values — field naming conventions, coordinate order, encoding
- For spatial data: you need to know the CRS before any spatial operations

### Detecting the CRS

Knowing the CRS is essential before any spatial operation. The method depends on how geometry is stored:

**Step 1: Identify the format.** Run DESCRIBE first — the column type tells you which case you're in:

| Column type | Format | CRS location |
|---|---|---|
| `geometry('epsg:4326')` | GeoParquet or native Parquet geometry | CRS in column type (spatial resolves EPSG) |
| `geometry` (no CRS) | Native Parquet geometry with no CRS, or CRS not resolved | Check `parquet_schema()` logical_type |
| `blob` | Plain Parquet with WKB | No CRS embedded — must know from context |

**Case 1: GeoParquet (Parquet with `geo` KV metadata)**

```sql
-- Quickest: DESCRIBE shows CRS in the column type (requires LOAD spatial for EPSG resolution)
LOAD spatial;
DESCRIBE FROM 'data.parquet';
-- → geom  geometry('epsg:4326')

-- Or use ST_CRS on a row
SELECT ST_CRS(geom) FROM 'data.parquet' LIMIT 1;
-- → 'EPSG:4326'

-- Without spatial loaded, DESCRIBE still works but shows raw PROJJSON instead of the short code:
-- → geom  geometry('{"$schema":"https://proj.org/schemas/v0.5/projjson.schema.json",...}')

-- Full GeoParquet metadata inspection (no spatial extension needed):
WITH kv AS (
    SELECT file_name,
           decode(key) AS key_str,
           decode(value)::JSON AS geo_json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yharby/duckdb-skill](https://github.com/yharby/duckdb-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
