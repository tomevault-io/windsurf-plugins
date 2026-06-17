---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a WordPress site export/import system that enables resumable, cursor-based synchronization of both database content and filesystem data over HTTP. The system is designed to work on resource-constrained shared hosting environments by carefully managing memory and execution time.

## Core Architecture

The codebase follows a producer-consumer pattern with two main components:

### Export Side (Server) — `packages/reprint-exporter/src/`
- **export.php**: HTTP endpoint that serves as the export API, handling authentication and routing requests to the appropriate producer
- **MySQLDumpProducer**: Generates SQL dump fragments with cursor-based resumption, supporting batched INSERT statements and all MySQL data types
- **FileTreeProducer / FileListProducer**: Streams filesystem contents (full tree or explicit list) in chunks with support for symlinks and cursor-based resumption

### Import Side (Client) — `packages/reprint-importer/src/`
- **import.php**: CLI script that downloads from export.php using streaming multipart parsing, no buffering of entire response
- **MultipartStreamParser**: Incremental multipart/mixed parser that processes chunks as they arrive

### Supporting Classes
- **MysqlValueFormatter**: Formats MySQL values by type (NULL, numeric, binary hex, quoted strings)
- **ColumnTypeCache**: Queries and caches INFORMATION_SCHEMA.COLUMNS data
- **FileSnapshotStorage / SqliteSnapshotStorage**: Pluggable snapshot storage for deletion tracking

## Key Design Patterns

### Cursor-Based Reentrancy
Both producers support pausing and resuming via JSON cursors that encode complete state:
- Current table/file position
- Accumulated rows/chunks waiting to emit
- Last processed primary key values or byte offsets

Cursors are JSON strings internally, base64-encoded for HTTP transmission in X-Cursor (outgoing) and X-Export-Cursor (incoming) headers.

### Resource Budgeting
The system tracks memory and execution time limits to gracefully end requests before hitting host limits. This prevents process termination and allows resumption.

### Streaming Multipart Transport
Uses multipart/mixed content-type to split large files into chunks while transmitting per-chunk metadata (cursor, size, path). This allows splitting arbitrary-sized files across multiple HTTP requests.

## Development Commands

### Running Tests

```bash
# Run all PHPUnit tests
composer test

# Run with coverage (requires Xdebug)
composer test:coverage

# Run specific test file
cd tests && vendor/bin/phpunit MySQLDumpProducer/BasicDumpTest.php

# Run specific test method
cd tests && vendor/bin/phpunit --filter testRoundTripIntegrity
```

### Coding Standards

This repo uses WordPress Coding Standards for PHP. The ruleset lives in
`phpcs.xml.dist` and has temporary exclusions so cleanup can happen in focused
passes.

```bash
# Run WPCS
composer lint:php

# Apply PHPCS auto-fixes from the ruleset
composer lint:php:fix

# Run PHP 7.4+ compatibility
composer lint:php:compat
```

### Running E2E Tests

```bash
# From tests/e2e directory
cd tests/e2e

# Install JavaScript dependencies
npm install

# Run all end-to-end scenarios
npm run test

# Run a single scenario
npm run test -- tests/import-01-basic-file-sync.test.js
```

There are 49 E2E test files in `tests/e2e/tests/`, named `import-NN-description.test.js`. Each test spins up Docker containers with WordPress and runs a full import scenario.

### Database Configuration

Tests use environment variables defined in tests/phpunit.xml:
- DB_HOST (default: 127.0.0.1)
- DB_USER (default: root)
- DB_PASS (default: my-secret-pw)
- DB_NAME (default: test_mysql_dump)

Override with environment variables if needed.

## Important Implementation Details

### Symlink Security

Symlinks ARE automatically recreated during import. This is safe because all paths are relative to the `--fs-root` directory, preventing directory traversal outside it. Errors are logged to the audit log.

### Server-Side Directory Dedup

The file indexer (`endpoint_file_index` in `export.php`) prevents duplicate traversal of directories that overlap with configured roots. The `should_skip_index_root()` function checks each directory's `realpath()` against the scheduled root list — if a directory is a duplicate or parent of an already-scheduled root, traversal skips it. This is critical for WP.com Atomic sites where symlinks create overlapping paths (e.g. `/srv/htdocs/srv` → `/srv` creating infinite cycles, or `/wordpress/` and `/srv/htdocs/wordpress/` resolving to the same location).

### Non-Empty fs-root Handling (`--on-fs-root-nonempty`)

By default, `files-pull` refuses to start if `--fs-root` is non-empty (to prevent accidental overwrites). The `--on-fs-root-nonempty` flag controls this behavior:

- `--on-fs-root-nonempty=error` (default): throw an error and abort.
- `--on-fs-root-nonempty=preserve-local`: import into the non-empty directory while preserving all existing local content.

In `preserve-local` mode:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WordPress/reprint](https://github.com/WordPress/reprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
