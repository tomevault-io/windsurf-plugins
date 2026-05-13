---
trigger: always_on
description: These are **critical** configurations specific to this Dune dbt repository. Follow these rules for all models.
---


# dbt Best Practices

## Repository-Specific Configurations (Dune dbt Template)

These are **critical** configurations specific to this Dune dbt repository. Follow these rules for all models.

### Schema Configuration
- **NEVER declare `schema` property in model configs** (either in-model `config()` or in `dbt_project.yml`)
- Schema names are automatically handled by the custom `generate_schema_name` macro in `macros/dune_dbt_overrides/get_custom_schema.sql`
- Schema naming logic:
  - **Production** (`--target prod`): Uses `target.schema` from `profiles.yml` (set via `DUNE_TEAM_NAME` env var)
  - **Dev with suffix** (`--target dev` + `DEV_SCHEMA_SUFFIX` env var): `<team_name>__tmp_<suffix>`
  - **Dev without suffix** (`--target dev`, no env var): `<team_name>__tmp_`
- In CI workflows, `DEV_SCHEMA_SUFFIX` is automatically set to the PR number

### Alias Configuration
- **ALWAYS provide an `alias` config for every model**
- **Preference**: Declare alias in the model file itself using `{{ config(alias = 'model_name') }}` for transparency
- **Backup**: Can be assigned in `dbt_project.yml` if needed
- **Why**: Alias is how we differentiate tables/views since schema names are controlled by the custom macro
- Example:
  ```sql
  {{ config(
      alias = 'my_model_name'
      , materialized = 'view'
  ) }}
  ```

### Table Configuration
- **Required for all tables**: `on_table_exists: replace` is set globally in `dbt_project.yml`
- Do not override this setting
- **Why**: Dune's Hive metastore does **not** allow table renames, so standard dbt table operations (drop temp, create temp, rename existing, rename temp to final, drop backup) won't work
- File format defaults to `delta` (Dune Hive metastore default)
- Do not attempt to set `file_format` or `format` config - dbt fails on this property

### Source Configuration
- **Custom source macro override**: The `source()` macro automatically sets `database='delta_prod'` for all sources
- Macro location: `macros/dune_dbt_overrides/source.sql`
- Use sources normally with `{{ source('source_name', 'table_name') }}`
- **To override the default database**: Pass the `database` parameter directly in the source call
  - Example: `{{ source('source_name', 'table_name', database='custom_database') }}`

### Environment Variables
- **Required**: `DUNE_API_KEY` - Your Dune API key for authentication
- **Required**: `DUNE_TEAM_NAME` - Your team name, used as the base schema name (defaults to 'dune' if not set)
- **Optional**: `DEV_SCHEMA_SUFFIX` - Suffix for dev schema names (automatically set to PR number in CI)

### Connection Configuration (profiles.yml)
- **Do not change** `user: dune` - User is always 'dune'
- **Do not change** `catalog: dune` - Catalog is always 'dune'
- **Do not change** `host: dune-api-trino.dune.com` - Fixed Dune API endpoint
- Authentication via `DUNE_API_KEY` environment variable
- Connection type: Trino with LDAP method
- Session properties: `transformations: true` is required
- Timezone: UTC

## Development Workflow

Follow this recommended workflow when developing dbt models for Dune.

### 1. Start in the Dune App

**Always begin with a working query in the Dune web application before creating a dbt model.**

- Write and test your query logic in Dune's query editor first
- Ensure the data output matches your expectations
- Test data quality and edge cases
- Iterate quickly in the Dune app to validate your logic
- **Why**: The Dune app provides immediate feedback and is easier to debug than dbt runs

### 2. Convert Query to dbt Model

Once your query is working correctly, convert it to a dbt model.

**Choose the right materialization based on your use case:**

- **`view`** - For quick-running queries (< 1-2 minutes)
  - No data is stored, query runs each time the view is queried
  - Best for: Lightweight transformations, filters, joins on small datasets
  
- **`table`** - For full snapshot rebuilds
  - Entire table is recreated on each run
  - Best for: Medium-sized datasets, aggregations that need full refresh, dimension tables
  
- **`incremental`** - For large tables with time-based or event-based data
  - Only adds/updates recent rows instead of rebuilding entire table
  - Best for: Large fact tables, transaction logs, event streams
  - Requires careful configuration (see Incremental Model Configuration section)

**Always use `source()` and `ref()` Jinja functions:**
```sql
-- ✅ GOOD: Using source() and ref() for lineage
select
	t.block_time
	, t.hash
	, t.from
	, t.to
	, u.symbol
from
	{{ source('ethereum', 'transactions') }} as t
left join {{ ref('stg_user_addresses') }} as u
	on t.from = u.address
```

**Why**: Using `source()` and `ref()` enables:
- Automatic dependency resolution and execution order
- Complete data lineage tracking
- Impact analysis when models change

### 3. Use Short Date Filters During Development

**When developing with large datasets, always start with restricted date ranges to iterate faster.**

- Use short date filters (e.g., last 1-3 days) during initial development
- This speeds up iterations and reduces credit consumption
- For incremental models, use the `is_incremental()` conditional to force short filters on full refreshes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duneanalytics/dune-dbt-template](https://github.com/duneanalytics/dune-dbt-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
