---
trigger: always_on
description: > Self-updating protocol: any agent that changes this project must update this file before finishing.
---

# CLAUDE.md -- InfinitSpace Data Warehouse

> Self-updating protocol: any agent that changes this project must update this file before finishing.

---

## Project Overview

InfinitSpace Data Warehouse is a Python 3.11 Azure Functions ETL project that moves operational data into Azure SQL across these layers:

- `bronze`: raw source payloads
- `silver`: typed and normalized entities
- `ava`: denormalized product availability
- `core`: planned, not implemented

Primary sources today:

- Nexudus API
- Xero API
- CoStar PDF extractor (Real Estate HTTP function)
- Google Maps enrichment utilities exist but are not part of the scheduled Function App

Platform:

- Azure Functions
- Azure SQL
- Azure Blob Storage for raw Nexudus snapshots and Xero invoice PDFs
- Azure Storage Queue for silver fanout

Current deployment docs target:

- resource group: `infinitspace-prod-northeurope-data-rg`
- ETL app: `func-infinitspace-etl`
- storage account: `staccinfinitspaceprod001`

---

## Runtime Topology

Default ETL execution order in UTC:

1. `02:00` `nexudus_to_bronze`
2. `02:30` `bronze_to_silver`
3. queue fanout via `silver_entity_worker`
4. `03:00` `refresh_ava_availability`
5. `04:00` `xero_invoice_sync` (includes PDF caching for invoices missing `pdf_blob_path`)
6. `05:00` `bamboohr_sync`
7. `05:30` `replyio_stats_sync`

Important operational caveat:

- `bronze_to_silver` is schedule-based, not dependency-aware
- `refresh_ava_availability` is also schedule-based
- bronze should finish before silver starts
- silver workers should finish before AVA starts

Flow:

```text
Nexudus API
  -> nexudus_to_bronze
  -> bronze.nexudus_locations
  -> bronze.nexudus_products
  -> bronze.nexudus_contracts
  -> bronze.nexudus_coworker_invoices   (incremental via UpdatedSince watermark)
  -> bronze.nexudus_coworkers           (distinct CoworkerIds from invoices)
  -> bronze.nexudus_resources
  -> bronze.nexudus_extra_services
  -> blob snapshots (nexudus-raw-snapshots container)

bronze_to_silver
  -> Azure Storage Queue: silver-sync-tasks
  -> silver_entity_worker x 7
  -> silver.nexudus_locations
  -> silver.nexudus_location_hours
  -> silver.nexudus_products
  -> silver.nexudus_contracts
  -> silver.nexudus_coworker_invoices
  -> silver.nexudus_coworkers
  -> silver.nexudus_resources
  -> silver.nexudus_extra_services

refresh_ava_availability
  -> EXEC ava.sp_refresh_product_availability
  -> ava.product_availability

Xero API
  -> xero_invoice_sync
  -> bronze.xero_invoices
  -> silver.xero_invoices (pdf_blob_path populated when PDF is cached)
  -> silver.xero_invoice_line_items
  -> silver.xero_tenants
  -> xero.silver_tenants (view alias)
  -> optional bronze.xero_invoice_pdfs

Real Estate (HTTP trigger, optional)
  -> run_costar_extractor (HTTP POST /api/real-estate/costar/run)
  -> costar-extraction-tasks queue
  -> costar_extraction_worker
  -> downloads PDF from Blob (pdf-uploads/<blob_name>)
  -> BuildingContactExtractor (Anthropic API)
  -> uploads XLSX to Blob (excel-outputs/<job_id>_contacts.xlsx)
  -> updates bronze.costar_pdf_extractor_logs (Real Estate DB)

Reply.io API
  -> replyio_stats_sync (timer, 05:30 UTC)
  -> bronze.replyio_sequence_steps
  -> bronze.replyio_sequence_step_performance (daily stats, yesterday)
```

---

## Function App Registration Model

`function_app.py` registers functions based on app settings:

- `ENABLE_ETL_FUNCTIONS=1`
  - registers the production ETL surface
- `ENABLE_ADMIN_FUNCTIONS=1`
  - registers optional admin/debug HTTP routes

Default ETL deployment:

- `ENABLE_ETL_FUNCTIONS=1`
- `ENABLE_ADMIN_FUNCTIONS=0`
- `ENABLE_REAL_ESTATE_FUNCTIONS=0`

Optional admin deployment:

- `ENABLE_ETL_FUNCTIONS=0`
- `ENABLE_ADMIN_FUNCTIONS=1`

Optional Real Estate deployment (can be combined with ETL):

- `ENABLE_ETL_FUNCTIONS=1`
- `ENABLE_REAL_ESTATE_FUNCTIONS=1`
- `ENABLE_ADMIN_FUNCTIONS=0`

This means the default Azure Function App should show only:

- `nexudus_to_bronze`
- `bronze_to_silver`
- `silver_entity_worker`
- `refresh_ava_availability`
- `xero_invoice_sync`
- `replyio_stats_sync`

---

## Repository Structure

```text
Infinitspace-datawarehouse/
  CLAUDE.md
  README.md
  SQL_datawarehouse.md
  function_app.py
  host.json
  requirements.txt
  .env.example
  .funcignore
  functions/
    bronze_nexudus.py
    silver_nexudus.py
    silver_worker.py
    ava_refresh.py
    xero_sync.py
    integrations_admin.py
    admin_health.py
    real_estate_costar.py
    real_estate_costar_worker.py
    replyio_sync.py
  shared/
    azure_clients/
      sql_client.py
      bronze_writer.py
      blob_writer.py
      queue_client.py
      run_tracker.py
      silver_write_locations.py
      silver_writer_products.py
      silver_writer_contracts.py
      silver_writer_resources.py
      silver_writer_extra_services.py
      silver_writer_coworker_invoices.py
      silver_writer_coworkers.py
    nexudus/
      auth.py
      client.py
      transformers/
        locations.py
        products.py
        contracts.py
        resources.py
        extra_services.py
        coworker_invoices.py
        coworkers.py
    bamboohr/
      __init__.py
      client.py
      transformers/
        employees.py
    xero/
      oauth.py
      flow.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Infinitspace1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
