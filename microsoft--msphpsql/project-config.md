---
trigger: always_on
description: This document provides guidance for AI coding agents working with the msphpsql GitHub repository — the open-source home of the **sqlsrv** and **pdo_sqlsrv** PHP extensions for Microsoft SQL Server.
---

# AI Agent Guidelines for Microsoft Drivers for PHP for SQL Server

This document provides guidance for AI coding agents working with the msphpsql GitHub repository — the open-source home of the **sqlsrv** and **pdo_sqlsrv** PHP extensions for Microsoft SQL Server.

## Quick Start

### Essential Context Files

| File | Purpose |
|------|---------|
| [README.md](README.md) | Project overview, prerequisites, build basics |
| [CHANGELOG.md](CHANGELOG.md) | Full release history; current version at top |
| [Linux-mac-install.md](Linux-mac-install.md) | Step-by-step install guide for every supported Unix platform |
| [buildscripts/README.md](buildscripts/README.md) | Detailed Windows build instructions (manual & scripted) |
| [source/shared/version.h](source/shared/version.h) | Single source of truth for version numbers and preview state |

### Repository at a Glance

| Area | Path | Description |
|------|------|-------------|
| Shared core | `source/shared/` | C++ code shared by both extensions |
| SQLSRV extension | `source/sqlsrv/` | Procedural PHP API extension |
| PDO_SQLSRV extension | `source/pdo_sqlsrv/` | PDO interface extension |
| Packaging script | `source/packagize.sh` | Copies `shared/` into each extension dir for PECL packaging |
| Functional tests | `test/functional/` | `.phpt` test suites for both extensions |
| Extended tests | `test/extended/` | Always Encrypted v2 special tests |
| BVT tests | `test/bvt/` | Basic verification tests |
| Performance tests | `test/Performance/` | Benchmark suite (phpbench) |
| Windows build scripts | `buildscripts/` | `builddrivers.py`, `buildtools.py` |
| Samples | `sample/` | `sqlsrv_sample.php`, `pdo_sqlsrv_sample.php` |
| CI (Linux/Mac) | [azure-pipelines.yml](azure-pipelines.yml) | Azure Pipelines — builds PHP from source, runs tests |
| CI (Windows) | [appveyor.yml](appveyor.yml) | AppVeyor — PHP SDK build matrix |
| Docker | [Dockerfile-msphpsql](Dockerfile-msphpsql) | Ubuntu 24.04 dev environment image |

## Architecture

### Two Extensions, One Shared Core

The drivers ship as two separate PHP extensions that share a common C++ core layer:

```
source/
├── shared/            # Core logic (ODBC, connection, statement, stream, error handling)
│   ├── core_conn.cpp          # Connection management
│   ├── core_init.cpp          # Driver initialization
│   ├── core_results.cpp       # Result set processing
│   ├── core_stmt.cpp          # Statement execution
│   ├── core_stream.cpp        # Stream handling
│   ├── core_util.cpp          # Utility functions
│   ├── core_sqlsrv.h          # Main shared header (includes Zend API, ODBC, platform abstractions)
│   ├── version.h              # Version constants
│   ├── FormattedPrint.cpp/h   # Cross-platform printf implementation
│   ├── StringFunctions.cpp/h  # Cross-platform string operations
│   ├── localizationimpl.cpp   # Localization/encoding support
│   ├── msodbcsql.h            # ODBC driver header
│   └── xplat_*.h, typedefs_for_linux.h, interlockedatomic*.h  # Platform abstractions
│
├── sqlsrv/            # Procedural API extension
│   ├── conn.cpp       # sqlsrv_connect(), sqlsrv_close(), etc.
│   ├── init.cpp       # Extension module init/shutdown
│   ├── stmt.cpp       # sqlsrv_query(), sqlsrv_fetch(), etc.
│   ├── util.cpp       # Error handling, logging
│   ├── config.m4      # Linux/Mac autoconf build
│   └── config.w32     # Windows JScript build
│
├── pdo_sqlsrv/        # PDO extension
│   ├── pdo_dbh.cpp    # PDO database handle (new PDO(...))
│   ├── pdo_init.cpp   # Extension module init/shutdown
│   ├── pdo_parser.cpp # SQL parser for named params
│   ├── pdo_stmt.cpp   # PDOStatement operations
│   ├── pdo_util.cpp   # Error handling
│   ├── config.m4      # Linux/Mac autoconf build
│   └── config.w32     # Windows JScript build
│
└── packagize.sh       # Copies shared/ into sqlsrv/ and pdo_sqlsrv/ for PECL
```

### Key Design Patterns

- **Zend API integration**: All C++ code uses the PHP Zend Engine API. The shared header `core_sqlsrv.h` includes `<php.h>` and `<zend.h>` with MSVC warning suppression for known benign warnings (4100, 4127, 4146, 4244, 4267, 4456, 4457, 4706).
- **Platform abstraction**: Unix builds use `typedefs_for_linux.h`, `xplat_*.h`, and `interlockedatomic*.h` to map Win32 types/functions to POSIX equivalents (e.g., `stricmp` → `strcasecmp`, `GetLastError` → `errno`).
- **ODBC layer**: All SQL Server communication goes through the Microsoft ODBC Driver for SQL Server (version 17 or 18). The code never talks to TDS directly.
- **MARS**: Multiple Active Result Sets enabled by default via `MARS_Connection={Yes}`.

## Build System

### Linux / macOS (autoconf)

Each extension has a `config.m4` that registers the extension with PHP's build system:

```bash
# Build from PHP source tree (extensions in ext/ directory)
phpize
./configure --with-sqlsrv  # or --with-pdo_sqlsrv
make
make install
```

The `config.m4` includes logic to avoid double-compiling shared sources when both extensions are built as static (non-shared) in the same PHP build.

### Windows (PHP SDK + JScript)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/msphpsql](https://github.com/microsoft/msphpsql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
