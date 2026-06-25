---
trigger: always_on
description: **FHIR for Data Science** - A unified suite of high-performance tools for working with FHIR data in analytical environments, built on DuckDB.
---

# duckdb-fhirpath - AGENTS.md

**FHIR for Data Science** - A unified suite of high-performance tools for working with FHIR data in analytical environments, built on DuckDB.

## Repository Overview

This repository has been reorganized into a unified `fhir4ds` namespace. All Python source code resides under the `fhir4ds/` directory, organized by feature and backend.

## Package Structure (Unified)

| Feature | Subpackage Path | Purpose |
|---------|-----------------|---------|
| **FHIRPath** | `fhir4ds.fhirpath` | Core FHIRPath parser and evaluator |
| **FHIRPath (DuckDB)** | `fhir4ds.fhirpath.duckdb` | DuckDB integration and C++ extension wrapper |
| **CQL** | `fhir4ds.cql` | CQL to SQL translator for clinical quality measures |
| **CQL (DuckDB)** | `fhir4ds.cql.duckdb` | CQL-specific DuckDB UDFs and macros |
| **ViewDefinition** | `fhir4ds.viewdef` | SQL-on-FHIR v2 ViewDefinition support |
| **DQM** | `fhir4ds.dqm` | Digital Quality Measure orchestrator |

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      APPLICATION LAYER                          │
│  CQL Measures  │  FHIRPath Queries  │  ViewDefinitions         │
└────────┬───────────────┬───────────────────┬───────────────────┘
         │               │                   │
         ▼               ▼                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                      TRANSLATION LAYER                          │
│  fhir4ds.cql  │  (direct)          │  fhir4ds.viewdef          │
│  CQL → SQL    │                    │  ViewDef → SQL            │
└────────┬───────────────┴───────────────────┴───────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                      UDF LAYER (DuckDB)                         │
│  fhir4ds.fhirpath.duckdb     │  fhir4ds.cql.duckdb             │
│  fhirpath(), fhirpath_text() │  AgeInYears(), DurationInDays() │
└────────┬───────────────────────┴────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                      CORE LAYER                                 │
│  fhir4ds.fhirpath                                               │
│  FHIRPath parser and evaluator engine                           │
└─────────────────────────────────────────────────────────────────┘
```

## Subpackage Details

### `fhir4ds.fhirpath`
**Purpose:** Core FHIRPath parser and evaluator.
**Location:** `fhir4ds/fhirpath/`
**Tests:** `fhir4ds/fhirpath/tests/unit/`

### `fhir4ds.fhirpath.duckdb`
**Purpose:** Native DuckDB integration.
**Location:** `fhir4ds/fhirpath/duckdb/`
**Bundled Extension:** `fhir4ds/fhirpath/duckdb/extensions/fhirpath.duckdb_extension`

### `fhir4ds.cql`
**Purpose:** CQL translator and measure evaluator.
**Location:** `fhir4ds/cql/`
**Compliance:** 100% of implemented features pass official CQL compliance.

### `fhir4ds.viewdef`
**Purpose:** SQL-on-FHIR v2 implementation.
**Location:** `fhir4ds/viewdef/`
**Compliance:** 100% compliance with ViewDefinition v2 specification.

---

## Official Compliance Testing

The project maintains a unified conformance suite for validating against official standards.

```bash
# Run all conformance tests (FHIRPath, CQL, ViewDef, DQM)
python3 conformance/scripts/run_all.py
```

Reports are generated in `conformance/reports/`.

### Pytest Plugin Note

`pyproject.toml` intentionally sets `-p no:benchmark` in pytest `addopts`.
The auto-loaded `pytest-benchmark` plugin can hang pytest startup/collection in
this workspace. Do not remove that option unless the plugin issue has been
verified fixed; benchmark runs are handled by the scripts in `benchmarks/`.

### Test Skip / XFail Policy

Do not use `pytest.skip(...)` to hide translator, parser, or generator behavior
that is expected to work. Tests for supported behavior should fail normally.

Use `pytest.mark.xfail(..., raises=ExpectedError)` or an equivalent explicit
expected-error assertion when a spec case is intentionally invalid or a known
gap must remain visible. This keeps XPASS behavior visible when support lands.

Skips are appropriate for environment or fixture availability only, such as a
missing optional DuckDB/C++ extension build, missing external FHIR datasets, or
benchmark/conformance fixture submodules that are not present.

CMS integration tests that need include resolution should use
`fhir4ds/cql/tests/integration/helpers.py::make_cql_library_loader` so included
libraries such as `FHIRHelpers`, `QICoreCommon`, `Status`, `Hospice`, `SDE`,
`AHA`, and `AdultOutpatientEncounters` are exercised instead of skipped.

### Source Adapter SQL Safety

Source adapters must quote identifiers and string literals separately. Use
`quote_identifier()` for DuckDB identifiers and `quote_sql_literal()` for SQL
string literals; never interpolate paths, credentials, secret names, patient ids,
or connection strings directly into SQL. Cloud credential providers and option
keys are allowlisted in `fhir4ds/sources/filesystem.py`; extend that allowlist
deliberately with focused injection tests when adding provider options.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fhir4ds/fhir4ds](https://github.com/fhir4ds/fhir4ds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
