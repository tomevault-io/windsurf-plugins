---
trigger: always_on
description: Multi-tenant analytics platform. Customers (tenants) connect their data sources (ad platforms, ecommerce, analytics tools). We land their raw data, push it through a schema-enforced pipeline, and produce tenant-isolated star schemas for reporting.
---

# GATA Platform — Claude Code Context

## What This Is
Multi-tenant analytics platform. Customers (tenants) connect their data sources (ad platforms, ecommerce, analytics tools). We land their raw data, push it through a schema-enforced pipeline, and produce tenant-isolated star schemas for reporting.

## Architecture (5 Layers)

```
sources/{tenant}/{platform}/     → Raw data landing (auto-generated)
staging/{tenant}/{platform}/     → Schema hash routing + push to master models (auto-generated)
platform/master_models/          → Multi-tenant tables with raw_data_payload JSON column
intermediate/{tenant}/{platform}/ → Extract typed fields from JSON, tenant-filtered views
analytics/{tenant}/              → Star schema (facts + dims) via engine/factory macros
```

**Layers 1-3 are auto-generated** by `scripts/onboard_tenant.py`. Staging models use `sync_to_master_hub()` post-hook to MERGE data into master model tables.

**Layers 4-5 are hand-written.** Intermediate models extract fields from `raw_data_payload` JSON. Analytics models call factory macros that call engine macros.

## Key Patterns

### Engine/Factory Pattern (Shell Architecture)
- **Engines** (`macros/engines/{domain}/`): Source-specific SQL that reads from intermediate models and outputs a canonical column set. Example: `engine_facebook_ads_performance(tenant_slug)` reads `int_{tenant}__facebook_ads_facebook_insights` and outputs `tenant_slug, source_platform, report_date, campaign_id, ad_group_id, ad_id, spend, impressions, clicks, conversions`.
- **Factories** (`macros/factories/`): Config-driven engine discovery. Call `get_tenant_config(tenant_slug)` to find enabled sources, then dynamically resolve engine macros via `context.get('engine_' ~ source ~ '_performance')`. Example: `build_fct_ad_performance('tyrell_corp')` discovers facebook_ads, google_ads, instagram_ads from config and UNION ALLs their engine outputs.
- **Shell Models** (`models/analytics/{tenant}/`): Single-line SQL files that call a factory with only the tenant slug. Example: `{{ build_fct_ad_performance('tyrell_corp') }}`

### JSON Extraction (DuckDB Syntax)
All master models store raw data in a `raw_data_payload` JSON column. Intermediate models extract typed fields:
```sql
CAST(raw_data_payload->>'$.field_name' AS BIGINT)     -- scalar to type
raw_data_payload->>'$.field_name'                       -- scalar to VARCHAR
raw_data_payload->'$.nested_object'                     -- keep as JSON
```

### Push Circuit (sync_to_master_hub)
Staging models are views that SELECT from sources. A `post_hook` in `generate_staging_pusher` calls `sync_to_master_hub(master_model_id)` which does a MERGE INTO the master model table USING the staging view. The post-hook ensures the view exists before the MERGE fires. Master model tables must exist before staging models can push to them.

### Master Model Incremental Pattern
Master models use `materialized='incremental'` with `full_refresh=false` to retain historical data across runs. The mechanism:
1. **First run** → dbt creates the table (SELECT ... WHERE 1=0 produces the empty schema)
2. **Staging post-hooks** → `sync_to_master_hub()` MERGEs data in (deduped by tenant + platform + payload hash)
3. **Subsequent runs** → dbt sees table exists, runs incremental append (0 new rows), existing data untouched
4. **Staging post-hooks** → MERGE inserts only genuinely new rows

**`full_refresh=false`** means `dbt run --full-refresh` will NOT drop master models. This is intentional — master models are append-only sinks. To truly reset one, DROP the table manually in the warehouse.

**Safe refresh commands:**
```bash
# Refresh reporting layer only (most common)
dbt run --selector reporting_refresh

# Full refresh everything EXCEPT master models
dbt run --full-refresh --selector safe_full_refresh

# Inspect master models (rarely needed)
dbt run --selector master_models_only
```

### Staging Pusher (generate_staging_pusher)
The macro in `macros/onboarding/generate_staging_pusher.sql` creates staging views that:
1. SELECT from tenant source tables
2. Pack all source columns into a single `raw_data_payload` JSON column via `row_to_json(base)`
3. Fire `sync_to_master_hub()` as a `post_hook` (runs after view creation)

## Active Tenants

| Tenant | Ad Sources | Ecommerce | Analytics | Status |
|--------|-----------|-----------|-----------|--------|
| tyrell_corp | facebook_ads, instagram_ads, google_ads | shopify | google_analytics | Active |
| wayne_enterprises | bing_ads, google_ads | bigcommerce | google_analytics | Active |
| stark_industries | facebook_ads, instagram_ads | woocommerce | mixpanel | Active |

## Important File Locations

| What | Where |
|------|-------|
| dbt project root | `warehouse/gata_transformation/` |
| Tenant config | `tenants.yaml` (project root) |
| Connector catalog | `supported_connectors.yaml` (project root) |
| Engine macros | `warehouse/gata_transformation/macros/engines/{analytics,ecommerce,paid_ads}/` |
| Factory macros | `warehouse/gata_transformation/macros/factories/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thedatagata) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
