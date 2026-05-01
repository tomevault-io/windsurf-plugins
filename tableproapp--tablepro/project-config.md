---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TablePro is a native macOS database client (SwiftUI + AppKit) — a fast, lightweight alternative to TablePlus. macOS 14.0+, Swift 5.9, Universal Binary (arm64 + x86_64).

- **Source**: `TablePro/` — `Core/` (business logic, services), `Views/` (UI), `Models/` (data structures), `ViewModels/`, `Extensions/`, `Theme/`
- **Plugins**: `Plugins/` — `.tableplugin` bundles + `TableProPluginKit` shared framework. Built-in (bundled in app): MySQL, PostgreSQL, SQLite, ClickHouse, MSSQL, Redis, CSV, JSON, SQL export, MQL. Separately distributed via plugin registry: MongoDB, Oracle, DuckDB, Cassandra, Etcd, CloudflareD1, DynamoDB, BigQuery
- **C bridges**: Each plugin contains its own C bridge module (e.g., `Plugins/MySQLDriverPlugin/CMariaDB/`, `Plugins/PostgreSQLDriverPlugin/CLibPQ/`)
- **Static libs**: `Libs/` — pre-built `libmariadb*.a`, `libpq*.a`, etc. `Libs/ios/` — xcframeworks for iOS (Hiredis, LibPQ, MariaDB, OpenSSL, LibSSH2). Both downloaded from GitHub Releases via `scripts/download-libs.sh` (not in git)
- **SPM deps**: CodeEditSourceEditor (`main` branch, tree-sitter editor), Sparkle (2.8.1, auto-update), OracleNIO. Managed via Xcode, no `Package.swift`.

## Build & Development Commands

```bash
# Build (development) — -skipPackagePluginValidation required for SwiftLint plugin in CodeEditSourceEditor
xcodebuild -project TablePro.xcodeproj -scheme TablePro -configuration Debug build -skipPackagePluginValidation

# Clean build
xcodebuild -project TablePro.xcodeproj -scheme TablePro clean

# Build and run
xcodebuild -project TablePro.xcodeproj -scheme TablePro -configuration Debug build -skipPackagePluginValidation && open build/Debug/TablePro.app

# Release builds
scripts/build-release.sh arm64|x86_64|both

# Lint & format
swiftlint lint                    # Check issues
swiftlint --fix                   # Auto-fix
swiftformat .                     # Format code

# Tests
xcodebuild -project TablePro.xcodeproj -scheme TablePro test -skipPackagePluginValidation
xcodebuild -project TablePro.xcodeproj -scheme TablePro test -skipPackagePluginValidation -only-testing:TableProTests/TestClassName
xcodebuild -project TablePro.xcodeproj -scheme TablePro test -skipPackagePluginValidation -only-testing:TableProTests/TestClassName/testMethodName

# DMG
scripts/create-dmg.sh

# Static libraries (first-time setup or after lib updates)
scripts/download-libs.sh          # Download from GitHub Releases (skips if already present)
scripts/download-libs.sh --force  # Re-download and overwrite
```

### Updating Static Libraries

Static libs (`Libs/*.a`) are hosted on the `libs-v1` GitHub Release (not in git). When adding or updating a library:

```bash
# 1. Update the .a files in Libs/
# 2. Regenerate checksums
shasum -a 256 Libs/*.a > Libs/checksums.sha256
# 3. Recreate and upload the archive
tar czf /tmp/tablepro-libs-v1.tar.gz -C Libs .
gh release upload libs-v1 /tmp/tablepro-libs-v1.tar.gz --clobber --repo TableProApp/TablePro
# 4. Commit the updated checksums
git add Libs/checksums.sha256 && git commit -m "build: update static library checksums"

# iOS xcframeworks (Libs/ios/*.xcframework)
tar czf /tmp/tablepro-libs-ios-v1.tar.gz -C Libs/ios .
gh release upload libs-v1 /tmp/tablepro-libs-ios-v1.tar.gz --clobber --repo TableProApp/TablePro
```

## Architecture

### Plugin System

All database drivers are `.tableplugin` bundles loaded at runtime by `PluginManager` (`Core/Plugins/`):

- **TableProPluginKit** (`Plugins/TableProPluginKit/`) — shared framework with `PluginDatabaseDriver`, `DriverPlugin`, `TableProPlugin` protocols and transfer types (`PluginQueryResult`, `PluginColumnInfo`, etc.)
- **PluginDriverAdapter** (`Core/Plugins/PluginDriverAdapter.swift`) — bridges `PluginDatabaseDriver` → `DatabaseDriver` protocol
- **DatabaseDriverFactory** (`Core/Database/DatabaseDriver.swift`) — looks up plugins via `DatabaseType.pluginTypeId`
- **DatabaseManager** (`Core/Database/DatabaseManager.swift`) — connection pool, lifecycle, primary interface for views/coordinators
- **ConnectionHealthMonitor** — 30s ping, auto-reconnect with exponential backoff

Plugin bundles under `Plugins/`:

| Plugin                 | Database Types       | C Bridge             | Distribution |
| ---------------------- | -------------------- | -------------------- | ------------ |
| MySQLDriverPlugin      | MySQL, MariaDB       | CMariaDB             | Built-in     |
| PostgreSQLDriverPlugin | PostgreSQL, Redshift | CLibPQ               | Built-in     |
| SQLiteDriverPlugin     | SQLite               | (Foundation sqlite3) | Built-in     |
| ClickHouseDriverPlugin | ClickHouse           | (URLSession HTTP)    | Built-in     |
| MSSQLDriverPlugin      | SQL Server           | CFreeTDS             | Built-in     |
| RedisDriverPlugin      | Redis                | CRedis               | Built-in     |
| MongoDBDriverPlugin    | MongoDB              | CLibMongoc           | Registry     |
| DuckDBDriverPlugin     | DuckDB               | CDuckDB              | Registry     |
| OracleDriverPlugin     | Oracle               | OracleNIO (SPM)      | Registry     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TableProApp/TablePro](https://github.com/TableProApp/TablePro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
