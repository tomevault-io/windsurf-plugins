---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

django-litestream is a Django app that integrates [Litestream](https://litestream.io) (a SQLite replication tool) as Django management commands. The package automatically downloads the Litestream binary on first use and exposes all Litestream functionality through `python manage.py litestream` commands.

## Development Commands

### Package Management
```bash
# Install dependencies
just install
# or
uv sync

# Run tests
just test
# or
uv run pytest
```

### Demo Application
```bash
# Run Django commands in the demo app
just dj <command>
# Examples:
just dj migrate
just dj shell

# Run the demo server
just run-demo
```

### Code Quality
```bash
# Format code with ruff and prek
just fmt
```

### Release Process
```bash
# Bump version (patch, minor, or major) and update changelog
just bumpver patch
```

## Architecture

### Core Components

**Configuration System (src/django_litestream/conf.py)**
- `AppSettings` dataclass reads from `settings.LITESTREAM`
- Key settings: `path_prefix`, `bin_path`, `vfs_extension_path`, and all Litestream config options
- `path_prefix` is prepended to replica paths for multi-project bucket organization
- `bin_path` defaults to `<venv>/bin/litestream` but is auto-downloaded if missing

**Management Command (src/django_litestream/management/commands/litestream.py)**
- Single Django command that wraps all Litestream subcommands
- `generate_temp_config()` converts Django settings to temporary YAML config file
- Database alias resolution: users can specify "default" instead of full paths
- Auto-generates S3 replica config when not explicitly provided
- Binary auto-download on first use via `download_binary()`
- Custom `verify` subcommand (not part of upstream Litestream)

### Supported Commands

All upstream Litestream commands plus custom commands:

| Command | Description |
|---------|-------------|
| `databases` | List databases specified in config file |
| `ltx` | List available LTX files for a database |
| `replicate` | Runs a server to replicate databases |
| `restore` | Recovers database backup from a replica |
| `status` | Show replication status for databases |
| `sync` | Force immediate WAL-to-LTX sync |
| `version` | Prints the binary version |
| `config` | Show generated Litestream configuration (custom) |
| `verify` | Verify backup integrity (custom) |
| `vfs-install` | Download VFS extension (custom) |

### Configuration Translation

The command dynamically generates Litestream config from Django settings:

1. Reads `LITESTREAM` dict from Django settings
2. Resolves Django database aliases to actual file paths using `settings.DATABASES`
3. For each database without explicit `replica` config:
   - Auto-generates S3 replica pointing to `$LITESTREAM_REPLICA_BUCKET`
   - Applies `path_prefix` to the replica path
   - Adds global `access-key-id` and `secret-access-key` env var references
4. Writes YAML to temporary file
5. Passes config path to Litestream binary via `-config` flag

### Database Alias Resolution

`_db_location_from_alias()` function allows users to reference databases by Django alias:
- If alias exists in `settings.DATABASES` and uses SQLite engine, returns the `NAME` path
- Otherwise returns the original string (allows direct paths or replica URLs)
- Used for all `db_path` arguments across commands

### Verify Command

Custom integrity check (inspired by litestream-ruby):
1. Inserts unique verification row into `_litestream_verification` table
2. Waits 10 seconds for replication
3. Restores latest backup to temporary location
4. Verifies the verification row exists in restored database
5. Returns success/failure based on whether backup is in sync

### VFS (Virtual File System) Feature

The VFS feature enables read-only access to database replicas stored in cloud object storage without downloading the entire database file. Pages are fetched on-demand and cached in memory.

**Architecture:**
- **VFS Extension**: Compiled SQLite extension (.so/.dylib) that registers a custom VFS handler
- **VFS Loader** (`vfs.py`): Thread-safe module that ensures extension loads exactly once per process
- **Custom Database Backend**: `django_litestream/db/backends/sqlite_vfs.py` extends Django's SQLite backend
- **Configuration Helper**: `get_vfs_databases()` in `__init__.py` generates Django database configs

**How It Works:**
1. User calls `get_vfs_databases()` in settings.py to generate VFS database configurations
2. On Django startup, `AppConfig.ready()` loads the VFS extension once (if VFS databases configured)
   - Uses `ensure_vfs_loaded()` with thread-safe double-checked locking
   - Auto-downloads extension if missing
   - Registers VFS handler globally for this process
3. When Django opens a VFS database connection, the custom backend:
   - Calls `ensure_vfs_loaded()` as fallback (no-op if already loaded)
   - Sets `LITESTREAM_REPLICA_URL` environment variable from `OPTIONS['litestream_replica_url']`
   - Opens the database with `?vfs=litestream&mode=ro` URI parameter
4. SQLite uses the already-registered VFS to fetch pages from cloud storage on-demand

**Configuration:**
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tobi-De/django-litestream](https://github.com/Tobi-De/django-litestream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
