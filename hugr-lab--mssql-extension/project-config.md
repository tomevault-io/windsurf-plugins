---
trigger: always_on
description: DuckDB extension for SQL Server via a custom TDS protocol implementation (no FreeTDS/ODBC).
---

# mssql-extension Development Guidelines

DuckDB extension for SQL Server via a custom TDS protocol implementation (no FreeTDS/ODBC).

## Technology

- **Language**: C++17 (DuckDB extension standard)
- **DuckDB**: main branch, supports both stable (1.4.x) and nightly APIs via `src/include/mssql_compat.hpp`
- **TLS**: OpenSSL via vcpkg (statically linked, symbol visibility controlled)
- **Platforms**: Linux (GCC), macOS (Clang), Windows (MSVC, MinGW/Rtools 4.2)

## Project Structure

```text
src/
  azure/                # Azure AD authentication infrastructure
  catalog/              # DuckDB catalog integration + transaction manager
  connection/           # Connection pooling, provider, settings
  dml/                  # DML operations
    insert/             # INSERT (batched VALUES, OUTPUT INSERTED for RETURNING)
    update/             # UPDATE (rowid-based, VALUES JOIN pattern)
    delete/             # DELETE (rowid-based, VALUES JOIN pattern)
  include/              # Headers (mirrors src/ layout)
  query/                # Query execution and result streaming
  table_scan/           # Table scan with filter/projection pushdown
  tds/                  # TDS protocol implementation
    auth/               # Authentication strategies (SQL auth, FEDAUTH)
    encoding/           # Type encoding (datetime, decimal, GUID, UTF-16)
    tls/                # TLS via OpenSSL with custom BIO callbacks
test/
  sql/                  # SQLLogicTest files (require SQL Server for most)
    attach/             # ATTACH/DETACH tests
    azure/              # Azure AD authentication tests (no SQL Server required)
    catalog/            # Catalog, DDL, filter pushdown, statistics
    copy/               # COPY TO MSSQL (BulkLoadBCP) tests
    ctas/               # CREATE TABLE AS SELECT tests
    dml/                # UPDATE and DELETE tests
    insert/             # INSERT tests
    integration/        # Core integration (pool, TLS, large data)
    query/              # Query-level tests
    rowid/              # Rowid pseudo-column tests
    tds_connection/     # TDS protocol tests
    transaction/        # Transaction management tests
  cpp/                  # C++ unit tests (no SQL Server required)
docs/                   # Architecture documentation (see docs/architecture.md)
docker/                 # SQL Server container and Linux CI build
```

## Versioning

- The extension version is defined in `CMakeLists.txt` as `MSSQL_EXTENSION_VERSION` (e.g., `set(MSSQL_EXTENSION_VERSION "0.1.10")`)
- This is passed to C++ code via the `MSSQL_VERSION` compile definition and returned by `mssql_version()`
- **When releasing a new version**: update `MSSQL_EXTENSION_VERSION` in `CMakeLists.txt` and `version` in `vcpkg.json`

## Commands

```bash
# Build
make                    # Release build (with TLS via vcpkg)
make debug              # Debug build
make clean              # Remove build artifacts

# Test
make test               # Unit tests (no SQL Server required)
make docker-up          # Start SQL Server container
make integration-test   # Integration tests (requires SQL Server)
make test-all           # All tests
make test-debug         # Tests with debug build

# Docker
make docker-up          # Start SQL Server test container
make docker-down        # Stop container
make docker-status      # Check container health

# Load extension in DuckDB CLI
./build/release/duckdb
# Or dynamically:
duckdb --unsigned -c "INSTALL mssql FROM local_build_debug; LOAD mssql;"
```

## Code Style

- C++17, follow DuckDB extension conventions
- Use clang-format (version 14): `find src -name '*.cpp' -o -name '*.hpp' | xargs clang-format -i`

## Naming Conventions

### Files
- Source files: `mssql_<component>.cpp` for extension code, `tds_<component>.cpp` for TDS protocol
- Headers mirror source layout in `src/include/`
- Test files: `test_<component>.cpp` (C++), `<feature>_<scenario>.test` (SQL)

### Namespace Prefix Rule

**Critical**: The `MSSQL` or `Tds` prefix depends on namespace placement:

| Namespace | Prefix | Rationale | Examples |
|-----------|--------|-----------|----------|
| `duckdb` (common) | **Required** (`MSSQL`, `Tds`) | Avoid name collisions in shared namespace | `MSSQLCatalog`, `MSSQLTransaction`, `TdsConnection`, `TdsSocket` |
| `duckdb::mssql` | **No prefix** | Already scoped by namespace | `ConnectionProvider`, `PoolStatistics`, `InsertConfig` |
| `duckdb::tds` | **No prefix** | Already scoped by namespace | `Connection`, `Socket`, `PacketType` |
| `duckdb::tds::tls` | **No prefix** | Already scoped by namespace | `TlsContext`, `TlsBio` |
| `duckdb::encoding` | **No prefix** | Already scoped by namespace | `TypeConverter`, `DatetimeEncoding` |

### Classes and Structs
- **PascalCase** always. Prefix per namespace rule above.
- Info/metadata structs: `PrimaryKeyInfo`, `ColumnInfo`, `TableMetadata`, `PoolStatistics`
- Config structs: `InsertConfig`, `DMLConfig`, `PoolConfig`

### Methods
- **PascalCase** (DuckDB convention): `GetConnection()`, `SetPinnedConnection()`, `ExecuteBatch()`
- Getters: `Get<Property>()` — Setters: `Set<Property>()`
- Boolean queries: `Is<State>()`, `Has<Property>()` (e.g., `IsAlive()`, `HasPinnedConnection()`)

### Variables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hugr-lab/mssql-extension](https://github.com/hugr-lab/mssql-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
