---
trigger: always_on
description: A Go CLI that benchmarks data insertion performance into a DuckLake catalog. Metadata can live in PostgreSQL (local or remote/managed, e.g. Supabase) or a local DuckDB file; Parquet data can live on the local filesystem or S3. Connections are resolved via `ConnectionConfig` (see `connection.go`), which supports three modes — see "Connection Modes" below.
---

# parkbench — DuckLake Streaming Benchmark Tool

A Go CLI that benchmarks data insertion performance into a DuckLake catalog. Metadata can live in PostgreSQL (local or remote/managed, e.g. Supabase) or a local DuckDB file; Parquet data can live on the local filesystem or S3. Connections are resolved via `ConnectionConfig` (see `connection.go`), which supports three modes — see "Connection Modes" below.

## Architecture

- **Metadata store**: PostgreSQL (local or remote, e.g. Supabase) or a local DuckDB `.ducklake` file
- **Data store**: Parquet files, either on local disk (default: `./ducklake_data/`) or S3 (`s3://bucket/prefix`)
- **Query engine**: DuckDB with the `ducklake` extension (via `github.com/duckdb/duckdb-go/v2`)
- **Catalog alias**: `wh` (how the catalog is referenced inside DuckDB SQL)

The tool uses DuckDB in-process (not a DuckDB server). Each command opens an in-memory DuckDB instance and `ATTACH`es to the DuckLake catalog via `openAndAttach()` in `connection.go`.

### Connection Modes

`ConnectionConfig` (in `connection.go`) resolves the `ATTACH` statement one of three ways:

1. **Named persistent secret** (`--ducklake-secret <name>`) — everything else is ignored; attaches via `ATTACH 'ducklake:<secret>' AS wh (...)`. The secret (created via `parkbench secrets create-ducklake` or the `duckdb` CLI) already bundles the Postgres connection, `DATA_PATH`, and metadata schema.
2. **Inline, local DuckDB** (`--metadata-store duckdb`) — a local `.ducklake` file plus a local data directory. Unchanged from the original implementation; zero-config local dev.
3. **Inline, Postgres** (`--metadata-store postgres`, default) — any Postgres via `--pg-dsn` (local or remote/managed like Supabase), writing to `--data-path` (a local directory or an `s3://bucket/prefix` URI). When the data path is S3 and no secret is set, parkbench creates a **temporary, non-persistent** `CREATE SECRET` for S3 credentials (`--s3-key-id`/`--s3-secret-key`/`--s3-region`, or `AWS_*` env vars) for that session only.

`ConnectionConfig.isRemote()` determines whether a connection points at infrastructure parkbench doesn't own outright (named secret, S3 storage, or non-localhost Postgres host) — this changes `reset` behavior (see below).

### ATTACH string formats

```sql
-- inline, local postgres (default)
ATTACH 'ducklake:postgres:dbname=ducklake_v1 host=localhost' AS wh
    (DATA_PATH './ducklake_data', AUTOMATIC_MIGRATION TRUE)

-- inline, remote postgres + s3, scoped to a metadata schema
ATTACH 'ducklake:postgres:host=db.xxxx.supabase.co port=5432 dbname=postgres user=postgres password=*** sslmode=require' AS wh
    (DATA_PATH 's3://bucket/prefix', AUTOMATIC_MIGRATION TRUE, METADATA_SCHEMA 'ducklake_meta')

-- named secret
ATTACH 'ducklake:ducklake_prod' AS wh (AUTOMATIC_MIGRATION TRUE, METADATA_CATALOG 'ducklake_prod_meta')
```

## DuckDB Secrets

Parkbench can create and consume DuckDB's persistent secrets (`CREATE PERSISTENT SECRET`), matching the pattern from [DuckLake in Production: Catalog and Storage](https://thefulldatastack.substack.com/p/ducklake-in-production-catalog-storage):

```bash
./parkbench secrets create-s3       --name s3_bucket   --key-id ... --secret-key ... --region us-east-1
./parkbench secrets create-postgres --name supabase_pg --host db.xxxx.supabase.co --database postgres --user postgres --password ...
./parkbench secrets create-ducklake --name ducklake_prod --data-path s3://bucket/prefix --metadata-secret supabase_pg --metadata-schema ducklake_meta
./parkbench secrets list
./parkbench secrets drop <name>
```

These are implemented in `secrets.go` and just execute the literal `CREATE PERSISTENT SECRET`/`DROP PERSISTENT SECRET` SQL via an ephemeral DuckDB connection — persistent secrets live in DuckDB's local secret store regardless of parkbench's process, so they're also visible/manageable from the plain `duckdb` CLI (`FROM duckdb_secrets();`).

## Prerequisites

For local Postgres mode (the default), PostgreSQL must be running and the catalog database must exist before any command is run:

```bash
brew services start postgresql@18
psql -d postgres -c "CREATE DATABASE ducklake_v1;"
```

> The Go tool does NOT start Postgres or create the database — that must be done manually as a one-time step. This is unnecessary for remote Postgres (Supabase, etc.), since the database already exists.

## Build

```bash
go build -o parkbench .
```

## Commands

### `setup` — one-time catalog initialization

Creates the `events`, `events_rich`, and `events_rejected` tables inside the DuckLake catalog. `CREATE TABLE IF NOT EXISTS` is used throughout, so `setup` is safe to re-run against an existing catalog (including a real production one).

```bash
./parkbench setup
./parkbench setup --pg-dsn "dbname=ducklake_v1 host=localhost" --data-path "./ducklake_data" --catalog wh
./parkbench setup --ducklake-secret ducklake_prod
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [early-signal-tech/parkbench](https://github.com/early-signal-tech/parkbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
