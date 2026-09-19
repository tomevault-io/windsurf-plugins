---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants working on the Hielo codebase.
---

# CLAUDE.md - Hielo Project Guide

This document provides comprehensive guidance for AI assistants working on the Hielo codebase.

## Project Overview

**Hielo** (Spanish for "ice") is a native desktop application for visualizing Apache Iceberg table metadata and snapshot history. Built with Rust and Dioxus, it provides a cross-platform GUI for exploring Iceberg tables from REST and AWS Glue catalogs.

### Key Features
- Multiple catalog support (REST and AWS Glue catalogs)
- Schema visualization with nested field support and schema evolution tracking
- Partition specification viewing with transform functions
- Snapshot timeline with filtering and table health analytics
- Persistent catalog configuration (stored in `~/.hielo/config.json`)

## Technology Stack

- **Language**: Rust (2024 edition)
- **UI Framework**: Dioxus 0.6 (desktop target)
- **Styling**: Tailwind CSS (loaded via CDN)
- **Iceberg Integration**: iceberg-rs 0.6.0 with REST and Glue catalog support
- **Async Runtime**: Tokio

## Project Structure

```
hielo/
├── src/
│   ├── main.rs           # Application entry point, main App component, navigation
│   ├── catalog.rs        # Catalog management (REST/Glue connections, namespace/table listing)
│   ├── catalog_ui.rs     # Catalog connection forms and table browser UI
│   ├── components.rs     # Table view components (Overview, Schema, Partitions, Snapshots)
│   ├── data.rs           # Core data types (IcebergTable, Schema, Snapshot, health metrics)
│   ├── analytics.rs      # Table health analytics engine (scoring, alerts, recommendations)
│   ├── config.rs         # Configuration persistence (~/.hielo/config.json)
│   └── iceberg_adapter.rs # Conversion from iceberg-rs types to internal types
├── Cargo.toml            # Rust dependencies and project config
├── Dioxus.toml           # Dioxus framework configuration
└── .github/workflows/    # CI/CD (build, test, cross-platform releases)
```

## Module Responsibilities

### `main.rs`
- Application state management (`AppState`, `AppTab`, `TableViewTab`)
- Main `App` component with tab-based navigation
- Left navigation pane with catalog/namespace/table tree
- Global search modal (Ctrl+K)
- Event handlers for table loading and tab management

### `catalog.rs`
- `CatalogManager` - manages catalog connections and queries
- `CatalogConfig` - configuration for REST/Glue catalogs
- `CatalogConnection` - active connection with catalog trait object
- Async methods: `connect_catalog`, `list_namespaces`, `list_tables`, `load_table`

### `catalog_ui.rs`
- `CatalogConnectionScreen` - initial connection screen
- `RestCatalogForm` / `GlueCatalogForm` - catalog-specific connection forms
- `TableBrowser` - namespace and table selection UI
- `SavedCatalogsSection` - quick connect to saved catalogs

### `components.rs`
- `TableOverviewTab` - table metadata and properties display
- `TableSchemaTab` - schema visualization with evolution comparison
- `TablePartitionsTab` - partition specification display
- `SnapshotTimelineTab` - snapshot history with filtering and health analytics
- Health analytics display components (`HealthScoreBadge`, `HealthCategoryCard`)

### `data.rs`
- `IcebergTable` - main table representation with all metadata
- `TableSchema`, `NestedField`, `DataType` - schema types
- `Snapshot`, `Summary` - snapshot information
- `PartitionSpec`, `PartitionField`, `PartitionTransform` - partitioning
- Health analytics types (`TableHealthMetrics`, `FileHealthMetrics`, etc.)

### `analytics.rs`
- `TableAnalytics::compute_health_metrics()` - main entry point for health analysis
- Health scoring based on industry best practices (Netflix, Salesforce, AWS)
- Alert generation for small files, high snapshot frequency, compaction needs
- Maintenance recommendations with priority and effort levels

### `config.rs`
- `AppConfig` - persistent configuration with catalog list
- Load/save to `~/.hielo/config.json`
- Catalog CRUD operations with uniqueness validation

### `iceberg_adapter.rs`
- `convert_iceberg_table()` - main conversion function
- Type conversions from iceberg-rs to internal representations
- Schema, snapshot, and partition spec conversion helpers

## Key Data Types

### Core Table Structure
```rust
IcebergTable {
    name: String,
    namespace: String,
    catalog_name: String,
    location: String,
    schema: TableSchema,
    schemas: Vec<TableSchema>,      // Historical schemas
    snapshots: Vec<Snapshot>,
    current_snapshot_id: Option<u64>,
    properties: HashMap<String, String>,
    partition_spec: Option<PartitionSpec>,
    partition_specs: Vec<PartitionSpec>, // Historical specs
}
```

### Catalog Types
```rust
enum CatalogType { Rest, Glue }

CatalogConfig {
    catalog_type: CatalogType,
    name: String,
    config: HashMap<String, String>, // uri, warehouse, region, etc.
}
```

## Development Commands

```bash
# Run in development mode
cargo run

# Build release binary
cargo build --release

# Run tests
cargo test

# Check formatting
cargo fmt --check

# Apply formatting
cargo fmt

# Run clippy lints
cargo clippy -- -D warnings

# Run all checks (what CI does)
cargo check --all-targets --all-features
cargo fmt --all -- --check
cargo clippy --all-targets --all-features -- -D warnings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atcol/hielo](https://github.com/atcol/hielo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
