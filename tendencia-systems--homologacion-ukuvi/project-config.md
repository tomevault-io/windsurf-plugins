---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a vehicle homologation system that unifies vehicle catalogs from multiple insurance companies (aseguradoras) into a single master catalog. The system extracts data from each insurer's database, normalizes it using n8n workflows, and stores it in a Supabase PostgreSQL database with intelligent deduplication powered by token-overlap matching.

## Architecture & Data Flow

### Core Components

1. **Data Sources**: Insurance company databases (Qualitas, HDI, AXA, GNP, Mapfre, Chubb, Zurich, Atlas, BX, El Potosí, ANA)
2. **n8n Workflows**: ETL processes for extraction, normalization, and batch processing
3. **Supabase Database**: PostgreSQL database with RPC functions for intelligent homologation
4. **Canonical Model**: Standardized vehicle representation with hash-based deduplication and token-overlap matching

### Data Flow Process

```
Insurance DB → SQL Extraction → n8n Normalization → Batch Processing → Supabase RPC → Master Catalog
```

## Key Architecture Patterns

### Hash-Based Deduplication with Token Overlap

- **`hash_comercial`**: SHA-256 of `marca|modelo|anio|transmision` for basic vehicle identification
- **Token comparison**: `tokenize_version(version)` generates normalized arrays for order-independent matching across insurers
- **Similarity thresholds**: Configurable ratios (typically ≥0.92 for same-insurer reprocesses and ≥0.50 for cross-insurer merges)

### Canonical Data Model

The master table `catalogo_homologado` follows this schema:

- **Core fields**: `marca`, `modelo`, `anio`, `transmision`
- **Integrated version**: `version` field contains ALL technical specifications as a single string
- **Availability tracking**: JSONB field `disponibilidad` with per-insurer active/inactive status
- **Traceability**: `string_comercial`, `string_tecnico` for debugging and audit
- **No separate technical fields**: All specs (power, displacement, cylinders, doors, traction) are embedded in the `version` string

### Active/Inactive Status Management

- Each insurer can mark vehicles as active/inactive in their availability data
- Global validity: vehicle is considered active if ANY insurer reports it as active
- No records are deleted; inactivation updates the `disponibilidad` JSONB field
- Reactivation is supported by updating `activo=true`

## Directory Structure

### `/src/insurers/[name]/`

Each insurer has dedicated files:

- `[name]-analisis.md`: Data profiling and field mapping analysis
- `[name]-query-de-extraccion.sql`: SQL extraction query
- `[name]-codigo-de-normalizacion.js`: n8n normalization logic (when applicable)
- `ETL - [Name].json`: Complete n8n workflow definition

### `/src/supabase/`

3. **Normalization Code Node**: JavaScript transformation using insurer-specific logic
4. **Batch Processing**: Split data into chunks (10k-50k records)
5. **Supabase RPC Call**: POST to `/rest/v1/rpc/procesar_batch_homologacion`

### Normalization Requirements

Each normalization script must:

- Generate `hash_comercial` (SHA-256 of marca|modelo|anio|transmision)
- Generate `id_canonico` (SHA-256 of complete record including version)
- Create integrated `version` field containing ALL technical specifications in a single string
- Format: `[TRIM] [BODY] [POWER] [DISPLACEMENT] [CYLINDERS] [DOORS] [TRACTION]`
- Example: `"ADVANCE SEDAN 145HP 2L 4CIL 4PUERTAS AWD"`
- Map transmission codes to standard values (MANUAL/AUTO/null)
- Remove comfort/security features (AA, EE, CD, ABS, BA) from version strings
- Convert door format from "4P" to "4PUERTAS"
- Apply consistent marca/modelo standardization
- Preserve original data for traceability

## Supabase RPC Interface

### Main Function: `procesar_batch_homologacion`

**Endpoint**: `/rest/v1/rpc/procesar_batch_homologacion`
**Input Schema**:

```json
{
  "body": {
    "vehiculos_json": [
      {
        "id_canonico": "string",
        "hash_comercial": "string",
        "string_comercial": "string",
        "string_tecnico": "string",
        "marca": "string",
        "modelo": "string",
        "anio": integer,
        "transmision": "MANUAL|AUTO|null",
        "version": "string - integrated technical specs",
        "origen_aseguradora": "string",
        "id_original": "string",
        "version_original": "string",
        "activo": boolean
      }
    ]
  }
}
```

### Processing Logic

1. **Staging**: Load records into temp table with deduplication
2. **Exact Match**: By `id_canonico` → updates availability only
3. **Token Match**: Same `hash_comercial` + overlap ratio on `version_tokens_array`
4. **Similarity thresholds**: Records with scores ≥ 0.92 (same insurer) or ≥ 0.50 (cross insurer) are considered matches
5. **New Record**: Creates new entry when no overlap exceeds thresholds
6. **Conflict Detection**: Reports low-overlap candidates for manual review
   **Returns**: Processing metrics including counts of new/updated/matched records plus warnings/errors

## Token Overlap Strategy

### PostgreSQL Helpers

- **`tokenize_version`**: PL/pgSQL helper that lowercases, strips punctuation, and deduplicates tokens

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TendencIA-Systems) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
