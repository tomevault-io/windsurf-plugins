---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a DuckDB extension for reading AnnData (.h5ad) files, which are the standard format for single-cell genomics data. The extension provides SQL access to annotated data matrices containing cells (observations) and genes (variables).

## Architecture

The extension maps AnnData components to DuckDB tables:
- `X` (expression matrix) → `main` table with (obs_id, var_id, var_name, value)
- `obs` (cell metadata) → `obs` table with observation metadata
- `var` (gene metadata) → Multiple tables: scalar columns in `var`, array columns in `var_<column>`
- `obsm`/`varm` (dimensional reductions) → Separate tables per matrix: `obsm_pca`, `obsm_umap`, etc.
- `layers` (alternative matrices) → Separate tables: `layers_raw`, `layers_normalized`, etc.
- `uns` (unstructured data) → Multiple approaches: `uns_scalar`, `uns_<dataframe>`, `uns_json`

## Key Design Decisions

1. **Read-only access**: Initial implementation focuses on reading AnnData files only
2. **Flexible gene/cell identifiers**: Support both IDs (e.g., Ensembl) and names (e.g., gene symbols) via configurable columns
3. **Variable-length array handling**: Arrays in var DataFrame decomposed into separate tables with (var_id, index, value) structure
4. **Sparse matrix optimization**: Automatic detection and efficient handling of sparse matrices
5. **Mirror SQLite extension**: Follow similar ATTACH/DETACH paradigm for familiarity

## SQL Interface

### Core Commands
```sql
-- Attach AnnData file
ATTACH 'data.h5ad' AS pbmc (TYPE ANNDATA);
ATTACH 'data.h5ad' AS pbmc (TYPE ANNDATA, VAR_NAME_COLUMN='gene_symbols', VAR_ID_COLUMN='ensembl_id');

-- Query data
SELECT * FROM pbmc.main WHERE var_name = 'CD3D';
SELECT * FROM pbmc.obs WHERE cell_type = 'T cell';
SELECT * FROM pbmc.obsm_umap;

-- Detach
DETACH pbmc;
```

### Configuration
```sql
SET anndata_chunk_size = 10000;
SET anndata_sparse_optimization = true;
SET anndata_cache_size = '1GB';
SET anndata_default_var_name_column = 'gene_symbols';
```

## Implementation Structure

Expected directory layout:
```
anndata_duckdb/
├── src/
│   ├── anndata_extension.cpp      # Main extension entry point
│   ├── anndata_attach.cpp          # ATTACH/DETACH implementation
│   ├── anndata_scanner.cpp         # Table function implementations
│   ├── anndata_schema.cpp          # Schema discovery and mapping
│   └── anndata_types.cpp           # Type conversion utilities
├── include/
│   └── anndata_extension.hpp       # Public API
└── CMakeLists.txt
```

## Development Notes

- The extension uses HDF5 library for reading .h5ad files
- Leverage DuckDB's columnar storage and query optimization
- Support zero-copy where possible to minimize memory overhead
- Implement lazy loading - data read on-demand from HDF5
- Push WHERE clause predicates to HDF5 reading when possible
- Handle sparse matrices efficiently using CSR/CSC formats
- use the documents in the `spec/` folder to guide the development process
- keep the specs in the `spec/implementation-plan.md` up to date as design decisions change
- use `uv` for all python package management. Prefer `uv add` over `uv pip install` in order to manage the package list with pyproject.toml
- use `uv run python3` for all python actions

## Building from a Clean Environment

### Prerequisites

The build requires system-installed C/C++ libraries that are NOT managed by vcpkg in local dev (vcpkg.json is only used by CI):

```bash
apt-get install -y libhdf5-dev libcurl4-openssl-dev libssl-dev
```

You also need: `cmake`, `make`, `g++`, `git`, and `uv` (for Python tooling / running make targets).

### Submodule initialization

The repo has two git submodules (`duckdb/` and `extension-ci-tools/`) that must be populated before building. If these directories are empty:

```bash
git submodule init
git submodule update
```

### Building

The standard build command is:

```bash
uv run make          # builds the release target
```

This runs cmake configure + build via the Makefile in `extension-ci-tools/makefiles/duckdb_extension.Makefile`. The first build takes several minutes (it compiles all of DuckDB). Subsequent builds after source changes are incremental and fast.

For faster iteration when fixing compile errors, you can split configure and build:

```bash
# One-time configure:
rm -rf build/release
mkdir -p build/release
cmake -DEXTENSION_STATIC_BUILD=1 \
  -DDUCKDB_EXTENSION_CONFIGS="$(pwd)/extension_config.cmake" \
  -DCMAKE_BUILD_TYPE=Release \
  -S ./duckdb/ -B build/release

# Rebuild (run this after each source edit):
cmake --build build/release --parallel $(nproc)
```

This avoids re-running cmake configure on each iteration, which is the default behavior of `uv run make`.

### Build outputs

- `build/release/duckdb` — DuckDB CLI with the extension statically linked
- `build/release/test/unittest` — test runner binary
- `build/release/extension/anndata/anndata.duckdb_extension` — loadable extension

### Smoke test

```bash
./build/release/duckdb -c "SELECT anndata_version();"
```

### Running tests

```bash
# Run a single test file
build/release/test/unittest "test/sql/anndata_basic.test" --test-dir .


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [honicky/anndata-duckdb-extension](https://github.com/honicky/anndata-duckdb-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
