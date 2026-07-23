---
trigger: always_on
description: Provides the `#[wrappers_fdw]` attribute macro that generates:
---

# CLAUDE.md - AI Assistant Guide for Wrappers

## Project Overview

Wrappers is a development framework for PostgreSQL Foreign Data Wrappers (FDWs), written in Rust. It enables querying external data sources (APIs, databases, files) as if they were regular PostgreSQL tables. The project is maintained by [Supabase](https://supabase.com).

**Documentation**: https://fdw.dev/ | **API Docs**: https://docs.rs/supabase-wrappers

## Repository Structure

```
wrappers/
├── supabase-wrappers/           # Core FDW framework library (crates.io: supabase-wrappers)
├── supabase-wrappers-macros/    # Procedural macros (#[wrappers_fdw])
├── wrappers/                    # Native FDW implementations (PostgreSQL extension)
│   └── src/fdw/                 # Individual FDW implementations
├── wasm-wrappers/               # WebAssembly-based FDW implementations (separate workspace)
│   └── fdw/                     # Individual Wasm FDW crates
└── docs/                        # MkDocs documentation site
```

## Workspace Configuration

The project uses Cargo workspaces with the following structure:

- **Main workspace** (`Cargo.toml`): Contains `supabase-wrappers`, `supabase-wrappers-macros`, and `wrappers`
- **Wasm workspace** (`wasm-wrappers/fdw/Cargo.toml`): Separate workspace for Wasm FDWs (excluded from main)

**Rust version**: 1.88.0 (specified in `workspace.package`)
**pgrx version**: 0.16.1 (PostgreSQL extension framework)

## Key Components

### supabase-wrappers (Core Framework)

The core library providing the `ForeignDataWrapper` trait (`supabase-wrappers/src/interface.rs`):

```rust
pub trait ForeignDataWrapper<E: Into<ErrorReport>> {
    // Required methods
    fn new(server: ForeignServer) -> Result<Self, E>;
    fn begin_scan(&mut self, quals: &[Qual], columns: &[Column], sorts: &[Sort], limit: &Option<Limit>, options: &HashMap<String, String>) -> Result<(), E>;
    fn iter_scan(&mut self, row: &mut Row) -> Result<Option<()>, E>;
    fn end_scan(&mut self) -> Result<(), E>;

    // Optional methods for modification
    fn begin_modify(&mut self, options: &HashMap<String, String>) -> Result<(), E>;
    fn insert(&mut self, row: &Row) -> Result<(), E>;
    fn update(&mut self, rowid: &Cell, new_row: &Row) -> Result<(), E>;
    fn delete(&mut self, rowid: &Cell) -> Result<(), E>;
    fn end_modify(&mut self) -> Result<(), E>;

    // Optional methods for aggregate pushdown
    fn supported_aggregates(&self) -> Vec<AggregateKind>;
    fn supports_group_by(&self) -> bool;
    fn begin_aggregate_scan(&mut self, aggregates: &[Aggregate], group_by: &[Column], quals: &[Qual], options: &HashMap<String, String>) -> Result<(), E>;

    // Optional methods
    fn re_scan(&mut self) -> Result<(), E>;
    fn get_rel_size(...) -> Result<(i64, i32), E>;
    fn import_foreign_schema(...) -> Result<Vec<String>, E>;
    fn validator(options: Vec<Option<String>>, catalog: Option<Oid>) -> Result<(), E>;
}
```

### Key Data Types (interface.rs)

- `Cell`: Enum representing data values (Bool, I8-I64, F32-F64, String, Date, Timestamp, Json, Uuid, arrays)
- `Row`: Collection of column names and cells
- `Column`: Column metadata (name, number, type_oid)
- `Qual`: WHERE clause predicate (field, operator, value, use_or)
- `Sort`: ORDER BY specification
- `Limit`: LIMIT/OFFSET values

### supabase-wrappers-macros

Provides the `#[wrappers_fdw]` attribute macro that generates:
- `<name>_fdw_handler()` - FDW handler entry point
- `<name>_fdw_validator()` - Option validation function
- `<name>_fdw_meta()` - Metadata function

Usage:
```rust
#[wrappers_fdw(
    version = "0.1.0",
    author = "Supabase",
    website = "https://example.com",
    error_type = "MyFdwError"  // Required
)]
pub struct MyFdw { ... }
```

## Available FDWs

### Native FDWs (wrappers/src/fdw/)

| FDW | Feature Flag | Supports Write | Aggregate Pushdown |
|-----|--------------|----------------|--------------------|
| BigQuery | `bigquery_fdw` | Yes | Yes |
| ClickHouse | `clickhouse_fdw` | Yes | Yes |
| MySQL | `mysql_fdw` | Yes | Yes |
| Stripe | `stripe_fdw` | Yes | No |
| S3 Vectors | `s3vectors_fdw` | Yes | No |
| S3 | `s3_fdw` | No | No |
| Firebase | `firebase_fdw` | No | No |
| Airtable | `airtable_fdw` | No | No |
| Auth0 | `auth0_fdw` | No | No |
| AWS Cognito | `cognito_fdw` | No | No |
| DuckDB | `duckdb_fdw` | No | No |
| Apache Iceberg | `iceberg_fdw` | No | No |
| Logflare | `logflare_fdw` | No | No |
| Redis | `redis_fdw` | No | No |
| SQL Server | `mssql_fdw` | No | Yes |
| HelloWorld | `helloworld_fdw` | No (demo) | No |

### Wasm FDWs (wasm-wrappers/fdw/)

Cal.com, Calendly, Clerk, Cloudflare D1, HubSpot, Infura, Notion, Orb, Paddle, Shopify, Slack, Snowflake

## Development Workflows

### Prerequisites

```bash
# Install Rust toolchain
rustup install 1.88.0
rustup default 1.88.0

# Install pgrx
cargo install --locked cargo-pgrx --version 0.16.1

# Initialize pgrx with PostgreSQL
cargo pgrx init --pg15 /usr/lib/postgresql/15/bin/pg_config

# For Wasm development
cargo install --locked cargo-component --version 0.21.1
rustup target add wasm32-unknown-unknown
```

### Building

```bash
# Build native FDWs
cd wrappers
cargo build --features "native_fdws pg15"

# Build specific FDW
cargo build --features "stripe_fdw pg15"

# Build Wasm FDWs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [supabase/wrappers](https://github.com/supabase/wrappers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
