---
trigger: always_on
description: - **NEVER** create fallback data or use default values when real data is missing
---

# Landbruget.dk Pipeline Development Rules

## Core Principles

### 1. STRICT NO-FALLBACK POLICY
- **NEVER** create fallback data or use default values when real data is missing
- **ALWAYS** error immediately with clear, specific messages about missing data
- **NEVER** allow pipelines to continue processing with incomplete or synthetic data
- Real data integrity is paramount - fail fast and fail clearly

### 2. DATA VALIDATION FIRST
- **ALWAYS** validate data existence and quality before processing
- **ALWAYS** log detailed statistics about data (counts, geometry validity, etc.)
- **ALWAYS** error immediately if required data is missing or invalid
- Use descriptive error messages that explain exactly what real data is needed

### 3. FOLLOW EXISTING PATTERNS
- **ALWAYS** align new pipelines with successful existing patterns (e.g., field_area_analysis.py)
- **ALWAYS** use consistent spatial join patterns and table creation methods
- **ALWAYS** follow the established error handling and logging patterns
- Consistency across pipelines is critical for maintenance

## Technical Implementation Rules

### DuckDB Spatial Operations

#### SPATIAL_JOIN Operator (DuckDB Spatial v1.2.2+)
**Always optimize for the SPATIAL_JOIN operator** introduced in [DuckDB Spatial PR #545](https://github.com/duckdb/duckdb-spatial/pull/545). This provides massive performance improvements over the previous blockwise nested-loop joins.

```python
# ✅ CORRECT: Structure queries to trigger SPATIAL_JOIN operator
self.conn.execute("""
    CREATE OR REPLACE TABLE result_table AS
    SELECT f.*, s.soil_type
    FROM fields_table f
    LEFT JOIN soil_types_table s ON ST_Intersects(f.geom, s.geom)
""")

# ✅ Supported spatial predicates that trigger SPATIAL_JOIN:
# ST_Intersects, ST_Contains, ST_Within, ST_Touches, ST_Crosses, 
# ST_Overlaps, ST_Covers, ST_CoveredBy, ST_ContainsProperly, ST_Equals

# ❌ AVOID: Complex spatial conditions that prevent SPATIAL_JOIN
# WHERE ST_Intersects(f.geom, s.geom) AND ST_Area(f.geom) > 100
```

#### ST_Dump Syntax
```python
# ✅ CORRECT: DuckDB ST_Dump syntax
UNNEST(ST_Dump(geometry)).geom

# ❌ INCORRECT: This will fail in DuckDB
(ST_Dump(geometry)).geom
```

#### Optimizing for SPATIAL_JOIN
```python
# ✅ Best practices for SPATIAL_JOIN performance:
# 1. Use single spatial predicates in JOIN conditions
# 2. Prepare build side (smaller table) with clean geometries
# 3. Use ST_Dump for complex geometries before joining
# 4. Filter out NULL geometries before spatial joins

# Example: Optimize large spatial join
self.conn.execute("""
    -- Prepare build side (smaller table)
    CREATE OR REPLACE TABLE soil_clean AS
    SELECT soil_id, soil_type, 
           UNNEST(ST_Dump(geometry)).geom as geom
    FROM soil_raw 
    WHERE geometry IS NOT NULL AND ST_IsValid(geometry)
""")

self.conn.execute("""
    -- Spatial join will use SPATIAL_JOIN operator
    CREATE OR REPLACE TABLE fields_with_soil AS
    SELECT f.field_id, f.area_ha, s.soil_type
    FROM fields_spatial f
    LEFT JOIN soil_clean s ON ST_Intersects(f.geom, s.geom)
""")
```

### Geometry Validation Pattern
```python
# ✅ ALWAYS validate geometry before processing
geom_stats = self.conn.execute("""
    SELECT 
        COUNT(*) as total,
        COUNT(CASE WHEN geometry IS NOT NULL THEN 1 END) as has_geom,
        COUNT(CASE WHEN geometry IS NOT NULL AND ST_IsValid(geometry) THEN 1 END) as valid_geom
    FROM table_name
""").fetchone()

self.log.info(f"Geometry validation: {geom_stats[0]:,} total, {geom_stats[1]:,} with geometry, {geom_stats[2]:,} valid")

if geom_stats[1] == 0:
    raise ValueError("No records have geometry data")
if geom_stats[2] == 0:
    raise ValueError("No records have valid geometry - all geometries are invalid")
```

### Spatial Table Creation Pattern
```python
# ✅ Use this pattern for spatial table creation
self.conn.execute("""
    CREATE OR REPLACE TABLE spatial_table AS
    SELECT
        field_id,
        -- other columns --
        UNNEST(ST_Dump(
            CASE 
                WHEN ST_IsValid(geometry) THEN geometry
                ELSE ST_MakeValid(geometry)
            END
        )).geom as geom
    FROM source_table
    WHERE geometry IS NOT NULL
        AND (ST_IsValid(geometry) OR ST_MakeValid(geometry) IS NOT NULL)
""")

# ✅ ALWAYS validate results immediately
count = self.conn.execute("SELECT COUNT(*) FROM spatial_table").fetchone()[0]
if count == 0:
    raise ValueError("spatial_table is empty after processing - no valid geometries found")
```

### SQL Join Best Practices
```python
# ✅ ALWAYS use table aliases to avoid ambiguous column references
self.conn.execute("""
    CREATE OR REPLACE TABLE result_table AS
    SELECT
        f.field_id,  -- Explicit table alias
        f.area_ha,
        s.soil_type,
        pe.percolation_value
    FROM fields_table f
    LEFT JOIN soil_table s ON f.field_id = s.field_id
    LEFT JOIN percolation_effects pe ON f.field_id = pe.field_id
""")
```

### Error Handling Pattern
```python
# ✅ CORRECT: Fail immediately with clear error
try:
    # Process data
    result = process_data()
    if result.count == 0:
        raise ValueError("Processing failed - no valid data found")
except Exception as e:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Klimabevaegelsen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
