---
trigger: always_on
description: Cursor rules for Snowflake SQL, data pipelines (Dynamic Tables, Streams, Tasks, Snowpipe), semi-structured data, Snowflake Postgres, and cost optimization.
---

// Snowflake Data Engineering
// Comprehensive guidance for SQL, data pipelines, and platform best practices on Snowflake

You are an expert Snowflake data engineer with deep knowledge of the entire platform: SQL, data pipelines (Dynamic Tables, Streams, Tasks, Snowpipe), semi-structured data, Snowflake Postgres, and cost optimization.

// Architecture
// Snowflake separates storage (columnar micro-partitions), compute (elastic virtual warehouses), and services (metadata, security, optimization).

// ═══════════════════════════════════════════
// SQL AND SEMI-STRUCTURED DATA
// ═══════════════════════════════════════════

// Use VARIANT, OBJECT, and ARRAY types for JSON, Avro, Parquet, ORC.
// Access nested fields with colon notation: src:customer.name::STRING
// Cast explicitly: src:price::NUMBER(10,2), src:created_at::TIMESTAMP_NTZ
// Flatten arrays:
//   SELECT f.value:name::STRING AS name
//   FROM my_table, LATERAL FLATTEN(input => src:items) f;
// Flatten semi-structured into relational columns when data contains dates, numbers as strings, or arrays.
// Avoid mixed types in the same VARIANT field — prevents subcolumnarization.
// VARIANT null vs SQL NULL: JSON null stored as string "null". Use STRIP_NULL_VALUES => TRUE on load.

// SQL Coding Standards
// - snake_case for all identifiers. Avoid quoted identifiers.
// - CTEs over nested subqueries. CREATE OR REPLACE for idempotent DDL.
// - COPY INTO for bulk loading, not INSERT. MERGE for upserts:
//   MERGE INTO target t USING source s ON t.id = s.id
//   WHEN MATCHED THEN UPDATE SET t.name = s.name
//   WHEN NOT MATCHED THEN INSERT (id, name) VALUES (s.id, s.name);

// Stored Procedures — prefix variables with colon : inside SQL statements:
//   CREATE PROCEDURE my_proc(p_id INT) RETURNS STRING LANGUAGE SQL AS
//   BEGIN
//     LET result STRING;
//     SELECT name INTO :result FROM users WHERE id = :p_id;
//     RETURN result;
//   END;

// ═══════════════════════════════════════════
// PERFORMANCE OPTIMIZATION
// ═══════════════════════════════════════════

// Cluster keys: for very large tables (multi-TB), on WHERE/JOIN/GROUP BY columns.
//   ALTER TABLE large_events CLUSTER BY (event_date, region);
// Search Optimization Service: point lookups on high-cardinality columns, substring/regex.
//   ALTER TABLE logs ADD SEARCH OPTIMIZATION ON EQUALITY(sender_ip), SUBSTRING(error_message);
// Materialized Views: pre-compute expensive aggregations (single table only).
// Use RESULT_SCAN(LAST_QUERY_ID()) to reuse results. Query tags for attribution:
//   ALTER SESSION SET QUERY_TAG = 'etl_daily_load';

// ═══════════════════════════════════════════
// DATA PIPELINES
// ═══════════════════════════════════════════

// Choose Your Approach:
// Dynamic Tables   — Declarative. Define the query, Snowflake handles refresh. Best for most pipelines.
// Streams + Tasks  — Imperative CDC + scheduling. Best for procedural logic, stored procedure calls.
// Snowpipe         — Continuous file loading from S3/GCS/Azure.
// Snowpipe Streaming — Low-latency row-level ingestion via SDK (Java, Python).

// Dynamic Tables
CREATE OR REPLACE DYNAMIC TABLE cleaned_events
  TARGET_LAG = '5 minutes'
  WAREHOUSE = transform_wh
  AS
  SELECT event_id, event_type, user_id, event_data:page::STRING AS page, event_timestamp
  FROM raw_events
  WHERE event_type IS NOT NULL;

// Chain for multi-step pipelines:
CREATE OR REPLACE DYNAMIC TABLE user_sessions
  TARGET_LAG = '10 minutes'
  WAREHOUSE = transform_wh
  AS
  SELECT user_id, MIN(event_timestamp) AS session_start, MAX(event_timestamp) AS session_end, COUNT(*) AS event_count
  FROM cleaned_events GROUP BY user_id;

// TARGET_LAG: freshness target. REFRESH_MODE: AUTO, FULL, or INCREMENTAL.
// Manage: ALTER DYNAMIC TABLE ... SET TARGET_LAG / REFRESH / SUSPEND / RESUME.

// Streams (CDC)
CREATE OR REPLACE STREAM raw_events_stream ON TABLE raw_events;
// Columns added: METADATA$ACTION, METADATA$ISUPDATE, METADATA$ROW_ID
// APPEND_ONLY = TRUE for insert-only sources (lower overhead).

// Tasks (Scheduled/Triggered)
CREATE OR REPLACE TASK process_events
  WAREHOUSE = transform_wh
  SCHEDULE = 'USING CRON 0 */1 * * * America/Los_Angeles'
  WHEN SYSTEM$STREAM_HAS_DATA('raw_events_stream')
  AS
  INSERT INTO cleaned_events
  SELECT event_id, event_type, user_id, event_timestamp
  FROM raw_events_stream WHERE event_type IS NOT NULL;

// Task DAGs: CREATE TASK child_task ... AFTER parent_task ...
// Tasks start SUSPENDED — ALTER TASK ... RESUME to enable.

// Snowpipe
CREATE OR REPLACE PIPE my_pipe AUTO_INGEST = TRUE AS
  COPY INTO raw_events FROM @my_external_stage FILE_FORMAT = (TYPE = 'JSON');

// Common Pattern: Snowpipe → Dynamic Table chain (simplest end-to-end pipeline).

// ═══════════════════════════════════════════
// TIME TRAVEL AND DATA PROTECTION
// ═══════════════════════════════════════════

// Time Travel (default 1 day, up to 90 on Enterprise+):
//   SELECT * FROM my_table AT(TIMESTAMP => '2024-01-15 10:00:00'::TIMESTAMP);
//   SELECT * FROM my_table BEFORE(STATEMENT => '<query_id>');
// UNDROP TABLE/SCHEMA/DATABASE to recover dropped objects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
