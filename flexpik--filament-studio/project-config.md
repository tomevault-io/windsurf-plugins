---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Package Overview

Filament Studio (`flexpik/filament-studio`) is a Laravel package that provides a dynamic data model manager for Filament v5 using EAV (Entity-Attribute-Value) storage. Users create collections with custom fields at runtime — no migrations needed per collection.

## Commands

All commands run from the package root (`packages/flexpik/filament-studio/`).

```bash
# Run all tests (uses in-memory SQLite via Orchestra Testbench)
vendor/bin/pest

# Run a single test file
vendor/bin/pest tests/Unit/FieldTypes/Types/TextFieldTypeTest.php

# Run tests matching a name
vendor/bin/pest --filter="it resolves EAV cast"

# Run a specific test suite
vendor/bin/pest --testsuite=Unit
vendor/bin/pest --testsuite=Feature

# Format PHP (Pint lives in host project, not package — must use absolute path or Docker)
docker exec php83 /var/www/html/crud/vendor/bin/pint --dirty --format agent
```

Tests use Pest v4 with Orchestra Testbench. The `TestCase` base class (`tests/TestCase.php`) sets up SQLite in-memory, registers all Filament service providers, converts migration `.php.stub` files to temp `.php` files, and provides `authenticateUser()` for tests needing auth.

## Docker Environment

The project uses Docker containers for PHP execution. The primary container is `php83` (PHP 8.3).

```bash
# Run tests via Docker (required for coverage/mutation testing)
docker exec -w /var/www/html/crud/packages/flexpik/filament-studio -e XDEBUG_MODE=off php83 vendor/bin/pest --compact

# Run mutation testing (requires PCOV, installed in php83 container)
docker exec -w /var/www/html/crud/packages/flexpik/filament-studio -e XDEBUG_MODE=off php83 vendor/bin/pest --mutate --path=src/SomeFile.php

# Run pint via Docker
docker exec php83 /var/www/html/crud/vendor/bin/pint --dirty --format agent
```

- **Host PHP**: Herd Lite PHP 8.4 — works for running tests but has no coverage driver
- **Docker php83**: PHP 8.3 with PCOV installed — required for mutation testing (`pest --mutate`)
- `composer.json` has `platform.php: 8.3.25` set so vendor is compatible with both host and Docker
- Mutation testing uses `pestphp/pest-plugin-mutate` (not infection, which doesn't support Pest)

## Test Coverage

Mutation testing MSI targets ≥80% per module. Run `vendor/bin/pest --mutate --path=src/SomeFile.php` via Docker to check.

Three test suites configured in `tests/Pest.php`:

- **Unit** and **Feature** — use `TestCase` (SQLite in-memory, no Spatie Permission)
- **Integration** — uses `SpatieTestCase` which extends `TestCase` and additionally loads `spatie/laravel-permission` migrations. Use this suite for tests that need Spatie roles/permissions.

When creating new tests that need Spatie permissions, place them in `tests/Integration/` so they automatically get `SpatieTestCase`.

## Architecture

### EAV Storage Pattern

Data is stored across four core tables instead of per-collection tables:

- `studio_collections` — schema definitions (name, slug, tenant_id, settings)
- `studio_fields` — field definitions per collection (field_type, settings JSON, sort_order)
- `studio_records` — individual items (UUID, collection_id, tenant_id)
- `studio_values` — actual data with 6 typed columns: `val_text`, `val_integer`, `val_decimal`, `val_boolean`, `val_datetime`, `val_json`

The `EavCast` enum maps each field type to its storage column.

### Registries (Singleton Pattern)

Two registries manage extensible type systems:

- **FieldTypeRegistry** — 33 built-in field types across 9 categories (text, numeric, boolean, selection, datetime, file, relational, structured, presentation). Each type extends `AbstractFieldType` and implements `toFilamentComponent()`, `toTableColumn()`, `toFilter()`, and `settingsSchema()`.
- **PanelTypeRegistry** — 9 built-in panel types (Metric, List, TimeSeries, BarChart, LineChart, PieChart, Meter, Label, Variable). Each extends `AbstractDmmPanel` with an associated widget class.

Custom types are registered via the plugin API: `FilamentStudioPlugin::make()->fieldTypes([...])->panelTypes([...])`.

### Dynamic Resource System

`DynamicCollectionResource` is a single Filament resource that serves all collections. It resolves the collection from the `{collection_slug}` route parameter and dynamically builds forms, tables, and filters using three service classes:

- **DynamicFormSchemaBuilder** — generates Filament form components from field definitions, handling sections, conditional visibility, and validation
- **DynamicTableColumnsBuilder** — generates table columns from visible fields
- **DynamicFiltersBuilder** — generates table filters from filterable fields

### Query Layer

`EavQueryBuilder` constructs queries across the EAV structure, supporting filtering with `FilterGroup`/`FilterRule` tree logic (22 operators), sorting, pagination, and aggregates.

### Supporting Services

- **VariableResolver** — resolves tokens like `$CURRENT_USER`, `$CURRENT_TENANT`, `$NOW(+1 day)` in panel configs and filters
- **ConditionEvaluator** — evaluates conditional visibility/required/disabled states on form fields

### REST API


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flexpik/filament-studio](https://github.com/flexpik/filament-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
