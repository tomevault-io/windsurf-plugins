---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a DuckDB extension called `adbc` that integrates Arrow ADBC (Arrow Database Connectivity) with DuckDB. It's built using the DuckDB extension template. Familiarize yourself with the ADBC interface.

There is a checkout of a similar project under ./odbc-scanner which is the ODBC scanner for DuckDB extension. This adbc extension is modeled after that extension but uses the ADBC interface instead.

There is also a checkout of the Airport DuckDB extension under ./airport. The Airport extension integrates DuckDB with Apache Arrow Flight and demonstrates C++ code that can read Arrow record batches and return them to DuckDB. The docs are under ./airport/docs/README.md, but you're mostly interested in airport_take_flight.cpp.

There is also a checkout of the DuckDB postgresql extension under ./duckdb-postgres.  The postgresql extension integrates DuckDB with psotgres and demonstrates C++ code that can interact with the foreign postgresql tables.

## Extension Functions

The extension provides the following functions:

### Connection Management
- `adbc_connect(options)` - Connect to an ADBC data source. Returns a connection handle (BIGINT). Options can be passed as a STRUCT (preferred) or MAP.
  - **Required options:**
    - `driver` - Driver name, path to shared library, or path to manifest file (.toml)
  - **Optional options:**
    - `entrypoint` - Custom entry point function name
    - `search_paths` - Additional paths to search for driver manifests (colon-separated on Unix, semicolon on Windows)
    - `use_manifests` - Enable/disable manifest search (default: 'true'). Set to 'false' to only use direct library paths.
    - `secret` - Name of a DuckDB secret to use for connection parameters
    - Other options are passed directly to the ADBC driver
- `adbc_disconnect(handle)` - Disconnect from an ADBC data source. Returns true on success.

#### Secrets Support
The extension supports DuckDB secrets for storing connection credentials. Secrets are automatically looked up based on the `uri` option (scope matching) or can be explicitly referenced by name.

**Creating a secret:**
```sql
CREATE SECRET my_postgres (
    TYPE adbc,
    SCOPE 'postgresql://myhost:5432',
    driver 'postgresql',
    uri 'postgresql://myhost:5432/mydb',
    username 'user',
    password 'secret'
);
```

**Secret parameters:**
- `driver` - ADBC driver name or path
- `uri` - Connection URI passed to the driver
- `username` - Database username
- `password` - Database password (automatically redacted in logs)
- `database` - Database name
- `entrypoint` - Custom driver entry point
- `extra_options` - MAP of additional driver-specific options

**Using secrets:**
```sql
-- Automatic lookup by URI scope
SELECT adbc_connect({'uri': 'postgresql://myhost:5432/mydb'});

-- Explicit secret by name
SELECT adbc_connect({'secret': 'my_postgres'});

-- Override secret options with explicit values
SELECT adbc_connect({'secret': 'my_postgres', 'uri': 'postgresql://otherhost:5432/otherdb'});
```

#### Driver Manifest Support
The extension supports ADBC driver manifests, which allow referencing drivers by name instead of full paths. When `use_manifests` is enabled (default), the driver manager searches for manifests in these locations:

**macOS/Linux:**
1. `ADBC_DRIVER_PATH` environment variable (colon-separated paths)
2. `$VIRTUAL_ENV/etc/adbc/drivers` (if in a virtual environment)
3. `$CONDA_PREFIX/etc/adbc/drivers` (if in a Conda environment)
4. `~/.config/adbc/drivers` (Linux) or `~/Library/Application Support/ADBC/Drivers` (macOS)
5. `/etc/adbc/drivers`

**Windows:**
1. `ADBC_DRIVER_PATH` environment variable (semicolon-separated paths)
2. Registry: `HKEY_CURRENT_USER\SOFTWARE\ADBC\Drivers\{name}`
3. `%LOCAL_APPDATA%\ADBC\Drivers`
4. Registry: `HKEY_LOCAL_MACHINE\SOFTWARE\ADBC\Drivers\{name}`

A manifest file is a TOML file (e.g., `sqlite.toml`) containing driver metadata and the path to the shared library.

### Transaction Control
- `adbc_set_autocommit(handle, enabled)` - Enable or disable autocommit mode. When disabled, changes require explicit commit.
- `adbc_commit(handle)` - Commit the current transaction.
- `adbc_rollback(handle)` - Rollback the current transaction, discarding all uncommitted changes.

### Query Execution
- `adbc_scan(handle, query, [params := row(...)], [batch_size := N])` - Execute a SELECT query and return results as a table. Supports parameterized queries via the optional `params` named parameter. The optional `batch_size` parameter hints to the driver how many rows to return per batch (default: driver-specific, typically 2048). This is a best-effort hint that may be ignored by drivers that don't support it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Query-farm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
