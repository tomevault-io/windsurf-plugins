---
trigger: always_on
description: This file provides guidance to AI coding agents (e.g., Claude Code, Cursor, ChatGPT Codex, Gemini) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (e.g., Claude Code, Cursor, ChatGPT Codex, Gemini) when working with code in this repository.

While working on Apache Kvrocks, please remember:

- Always use English in code and comments.
- Only add meaningful comments when the code's behavior is difficult to understand.
- Add or update tests to cover externally observable behavior and regressions when you change or add functionality.
- Always run the formatter before submitting changes.
- For non-trivial behavior, storage format, replication, or cluster changes, first look for an existing issue, discussion, or mailing list context before implementing.

## Build and Development Commands

### Building

```bash
# Configure with Ninja when you want faster incremental builds.
# The default generator is Makefiles unless --ninja is specified.
./x.py build --ninja

# Build kvrocks and utilities
./x.py build                    # Build to ./build directory
./x.py build -j N               # Build with N parallel jobs
./x.py build --unittest         # Build with unit tests
./x.py build -DENABLE_OPENSSL=ON  # Build with TLS support
./x.py build --ninja            # Use Ninja build system
./x.py build --skip-build       # Only run CMake configure
./x.py build -DCMAKE_BUILD_TYPE=Debug  # Debug build

# Run a local server
./build/kvrocks -c kvrocks.conf

# Fetch dependencies
./x.py fetch-deps               # Fetch dependency archives
```

If the build directory was configured with Ninja, prefer incremental rebuilds like `cd build && ninja -j16 kvrocks` instead of re-running CMake.

### Testing

```bash
# Build and run C++ unit tests
./x.py build --unittest
./x.py test cpp

# Run Go integration tests
./x.py test go

# Re-run a specific Go test name.
# x.py test go currently forwards extra flags to "go test" but still runs "./...".
./x.py test go build -run TestKMetadata
```

### Lint

You must run the formatter and linters before submitting code changes. This ensures code quality and consistency across the project. It requires installing `clang-format`, `clang-tidy`, and `golangci-lint` locally. Please refer to the CONTRIBUTING.md for setup instructions.

```bash
# Format code (must pass before submitting)
./x.py format

# Check code format (fails if not formatted)
./x.py check format

# Run clang-tidy
./x.py check tidy

# Run golangci-lint for Go tests
./x.py check golangci-lint
```

## Architecture Overview

Apache Kvrocks is a distributed key-value NoSQL database compatible with the Redis protocol, using RocksDB as its storage engine.

### Core Structure

- **`src/server/`**: Main server orchestration, connection handling, and worker threads. The `Server` class manages the event loop, worker threads, and coordinates all components.
- **`src/storage/`**: RocksDB integration layer. Key classes:
  - `Storage`: Manages RocksDB instance, column families, and write batches
  - `Context`: Passes snapshot and batch between APIs for transactional consistency
- **`src/commands/`**: Redis protocol command implementations. Each command type has a corresponding `Commander` subclass.
- **`src/types/`**: Redis data structure implementations (String, Hash, List, Set, ZSet, Stream, etc.)
- **`src/cluster/`**: Cluster management, slot migration, and replication.
- **`src/search/`**: Full-text search and vector search (HNSW) implementation.
- **`src/config/`**: Server configuration parsing and management.
- **`src/cli/`**: Command-line interface utilities.
- **`src/common/`**: Shared utilities and helper functions.
- **`src/stats/`**: Statistics and metrics collection.

### Key Patterns

- **Column Families**: 8 column families are used - PrimarySubkey, Metadata, SecondarySubkey, PubSub, Propagate, Stream, Search, Index.
- **Command Registration**: Commands are registered via the `REDIS_REGISTER_COMMANDS` macro with flags like `kCmdWrite`, `kCmdReadOnly`, `kCmdBlocking`, etc.
- **Write Batch with Index**: Used for transactional mode to group writes before commit.
- **Worker Thread Model**: Libevent-based async I/O with dedicated worker threads.
- **Namespace Isolation**: Token-based multi-tenancy using the `__namespace` column family.

### Data Encoding

- `METADATA_ENCODING_VERSION=1` (default): Encodes 64-bit size and expire time in milliseconds.
- `METADATA_ENCODING_VERSION=0`: Legacy encoding.

Refer to https://kvrocks.apache.org/community/data-structure-on-rocksdb for more details.

## Coding Style and Naming Conventions

- C++ formatting follows `.clang-format` (Google-based, 2-space indent, 120-column limit, sorted includes).
- Use `.cc`/`.h` file extensions with `snake_case` filenames.
- Types use `PascalCase`; match existing patterns in nearby code.
- Use existing utilities and helper functions when possible; avoid reinventing the wheel.
- Go code should stay `gofmt`-clean and comply with `tests/gocase/.golangci.yml`.

## Testing Guidelines

You could provide Go tests for integration-level verification of command behaviors and C++ unit tests for internal logic. Focus on testing new features or bug fixes, and avoid adding tests that don't verify meaningful behavior changes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/kvrocks](https://github.com/apache/kvrocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
