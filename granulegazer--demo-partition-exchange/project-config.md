---
trigger: always_on
description: This is an Oracle Database 19c demonstration project showcasing partition exchange methodology for efficient data archival. The project uses interval partitioning, compression, and metadata-only partition exchange operations.
---

# GitHub Copilot Instructions

## Project Context

This is an Oracle Database 19c demonstration project showcasing partition exchange methodology for efficient data archival. The project uses interval partitioning, compression, and metadata-only partition exchange operations.

## Key Technical Requirements

### Oracle Version
- Target: Oracle Database 19.26
- Compatibility: Oracle 11g and higher
- Edition: Works with Enterprise editions

### Core Technologies
- **Partitioning**: Range partitioning with INTERVAL clause
- **Exchange Method**: `ALTER TABLE ... EXCHANGE PARTITION` with staging tables
- **Compression**: `COMPRESS BASIC` (available in all editions)
- **Indexes**: LOCAL indexes only (partition-aligned)

## Code Generation Guidelines

### 1. Table Creation

Always use `FOR EXCHANGE WITH TABLE` syntax for archive and staging tables:

```sql
-- CORRECT
CREATE TABLE sales_archive
FOR EXCHANGE WITH TABLE sales
PARTITION BY RANGE (sale_date)
INTERVAL (NUMTODSINTERVAL(1, 'DAY'))
(
    PARTITION sales_old VALUES LESS THAN (DATE '2000-01-01')
)
COMPRESS BASIC;

-- AVOID: Manual column definition
CREATE TABLE sales_archive (
    sale_id NUMBER,
    sale_date DATE,
    -- ... manual columns
);
```

### 2. Partition Naming

- Initial partition: `sales_old` (not `p_initial` or `p_archive_initial`)
- Boundary date: `DATE '2000-01-01'` (use DATE literal, not TO_DATE)
- Auto-created partitions: Oracle generates names automatically (e.g., `SYS_P12345`)

### 3. Index Definitions

All indexes must be LOCAL for partitioned tables:

```sql
-- CORRECT
CREATE INDEX idx_sales_date ON sales(sale_date) LOCAL;
CREATE INDEX idx_sales_customer ON sales(customer_id) LOCAL;
CREATE INDEX idx_sales_region ON sales(region) LOCAL;

-- AVOID: Global indexes
CREATE INDEX idx_sales_date ON sales(sale_date);  -- Missing LOCAL
```

### 4. Partition Exchange Pattern

Always use the two-step exchange with staging table:

```sql
-- Step 1: Move partition from source to staging
ALTER TABLE sales EXCHANGE PARTITION partition_name
WITH TABLE sales_staging_temp
INCLUDING INDEXES WITHOUT VALIDATION;

-- Step 2: Move from staging to archive
ALTER TABLE sales_archive EXCHANGE PARTITION partition_name
WITH TABLE sales_staging_temp
INCLUDING INDEXES WITHOUT VALIDATION;

-- Step 3: Drop empty partition from source
ALTER TABLE sales DROP PARTITION partition_name;
```

### 5. Error Handling and Logging

Use the autonomous logging procedure for tracking steps and errors:

```sql
-- Log informational message
prc_log_error_autonomous(
    p_table_name   => 'ARCHIVE_PROCEDURE',
    p_error_type   => 'I',
    p_trace_code   => step_number,
    p_sql_error_num => NULL,
    p_sql_error_msg => NULL,
    p_error_info1  => 'Step description',
    p_error_info2  => additional_info,
    p_user_id      => USER
);

-- Log error
prc_log_error_autonomous(
    p_table_name   => 'ARCHIVE_PROCEDURE',
    p_error_type   => 'E',
    p_trace_code   => step_number,
    p_sql_error_num => SQLCODE,
    p_sql_error_msg => SQLERRM,
    p_error_info1  => 'Error description',
    p_error_info2  => NULL,
    p_user_id      => USER
);
```

Include proper cleanup in exception handlers:

```sql
EXCEPTION
    WHEN OTHERS THEN
        -- Log the error
        prc_log_error_autonomous(
            p_table_name   => 'PROCEDURE_NAME',
            p_error_type   => 'E',
            p_trace_code   => v_step,
            p_sql_error_num => SQLCODE,
            p_sql_error_msg => SQLERRM,
            p_error_info1  => 'Error in procedure',
            p_error_info2  => NULL,
            p_user_id      => USER
        );
        
        -- Clean up staging table if exists
        BEGIN
            EXECUTE IMMEDIATE 'DROP TABLE sales_staging_temp';
        EXCEPTION WHEN OTHERS THEN NULL;
        END;
        ROLLBACK;
        RAISE;
END;
```

### 6. Logging Procedure Specification

**Procedure**: `prc_log_error_autonomous`

**Purpose**: Logs error and informational messages to ERROR_LOG_TABLE using an autonomous transaction.

**Parameters**:
- `p_table_name` - Name of the table or procedure related to the log entry
- `p_error_type` - Type of log entry ('I' for info, 'E' for error)
- `p_trace_code` - Trace code or step number for tracking
- `p_sql_error_num` - SQL error number, if applicable
- `p_sql_error_msg` - SQL error message, if applicable
- `p_error_info1` - Additional information (step, message, details)
- `p_error_info2` - Additional information (optional)
- `p_user_id` - User ID or context for the log entry

**Usage Pattern**:
```sql
-- Start of procedure
v_step := 1;
prc_log_error_autonomous('ARCHIVE_PROC', 'I', v_step, NULL, NULL, 'Starting archival', NULL, USER);

-- Before each major step
v_step := v_step + 1;
prc_log_error_autonomous('ARCHIVE_PROC', 'I', v_step, NULL, NULL, 
    'Exchanging partition', partition_name, USER);

-- After successful step
prc_log_error_autonomous('ARCHIVE_PROC', 'I', v_step, NULL, NULL, 
    'Exchange completed', v_count || ' records', USER);

-- In exception handler
prc_log_error_autonomous('ARCHIVE_PROC', 'E', v_step, SQLCODE, SQLERRM, 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/granulegazer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
