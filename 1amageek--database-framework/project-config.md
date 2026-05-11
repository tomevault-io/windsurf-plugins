---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Vision

**database-framework** is a **protocol-extensible, customizable index database** designed for the AI era, built on FoundationDB's transactional guarantees.

### Design Philosophy

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Protocol-Based Extensibility                     │
│                                                                      │
│   IndexKind (protocol)  ───▶  IndexMaintainer (protocol)            │
│        ↓                            ↓                                │
│   User defines           Framework executes                          │
│   WHAT to index          HOW to maintain                            │
└─────────────────────────────────────────────────────────────────────┘
```

1. **Protocol-Driven**: New index types can be added without modifying core code
2. **Composable**: Combine Vector + Graph + FullText in unified queries (Fusion API)
3. **Transactional**: All operations backed by FoundationDB's ACID guarantees
4. **AI-Native**: First-class support for embeddings, knowledge graphs, and RAG patterns

## Traits and Conditional Compilation

### Trait System

database-framework uses SPM traits to support multiple storage backends at compile time.

```
database-framework              →  storage-kit
  FoundationDB (default)        →    FoundationDB
  SQLite                        →    SQLite
  PostgreSQL                    →    PostgreSQL
```

| Trait | Facade Module | Storage Backend | Use Case |
|-------|---------------|-----------------|----------|
| `FoundationDB` (default) | `Database` | FDBStorage | Server-side distributed database |
| `SQLite` | `FDBite` | SQLiteStorage | On-device (iOS/macOS) |
| `PostgreSQL` | (none) | PostgreSQLStorage | Server-side RDBMS |

### Build and Test Commands

```bash
# Build with default trait (FoundationDB)
swift build

# Build with SQLite only (no libfdb_c required)
swift build --traits SQLite

# Build with PostgreSQL
swift build --traits PostgreSQL

# Run all tests (requires local FoundationDB running)
swift test

# Run FDBite tests only (no libfdb_c required)
swift test --traits SQLite --filter FDBiteTests

# Run PostgreSQL tests only
swift test --traits PostgreSQL --filter PostgreSQLTests

# Run a specific test file
swift test --filter DatabaseEngineTests.ScalarIndexKindTests

# Run a specific test function
swift test --filter "DatabaseEngineTests.ScalarIndexKindTests/testScalarIndexKindIdentifier"

# Build with release optimization
swift build -c release
```

**Prerequisites**:
- **FoundationDB trait**: FoundationDB must be installed and running locally. The linker expects `libfdb_c` at `/usr/local/lib`.
- **SQLite trait**: No external dependencies.
- **PostgreSQL trait**: PostgreSQL server must be running.

### Compile Flags

| Flag | Active When | Used In |
|------|-------------|---------|
| `FOUNDATION_DB` | `FoundationDB` trait enabled | DatabaseEngine, Database, DatabaseCLICore |
| `POSTGRESQL` | `PostgreSQL` trait enabled | TestSupport |

### Conditional Compilation Rules

**Source code**:
- `import FDBStorage` and all `FDBStorageEngine` usage must be wrapped in `#if FOUNDATION_DB`
- Index modules (ScalarIndex, VectorIndex, GraphIndex, etc.) depend only on `StorageKit`, not `FDBStorage` — no `#if` needed
- `DatabaseCLI` executable is inherently FoundationDB-only (uses `ClusterConnection`)

**Package.swift**:
- `FDBStorage` dependency must use `condition: .when(traits: ["FoundationDB"])`
- Each target that imports FDBStorage needs `swiftSettings: [.define("FOUNDATION_DB", .when(traits: ["FoundationDB"]))]`

### Architecture: Backend-Agnostic Index Layer

```
Database (FDB facade)     FDBite (SQLite facade)
    ↓                         ↓
    └─────────┬───────────────┘
              ↓
    DatabaseEngine + Index Modules  ← StorageKit only (backend-agnostic)
              ↓
         StorageKit (protocol)
              ↓
    ┌─────────┼─────────┐
    ↓         ↓         ↓
FDBStorage  SQLite  PostgreSQL   ← Conditional via traits
```

All index modules and the query planner operate through `StorageKit` abstractions. Backend-specific code exists only in:
- `DBConfiguration.swift` — `StorageBackend.fdb()` case
- `DBContainer.swift` — `FDBStorageEngine` initialization
- `ClusterConnection.swift` — FDB cluster discovery (CLI only)
- `Database.swift` — `@_exported import FDBStorage` re-export

## Dependencies

### database-kit

**database-kit は安定版のため、GitHub URL を使用すること。**

```swift
// ✅ 正しい: GitHub URL を使用
.package(url: "https://github.com/1amageek/database-kit.git", branch: "main"),

// ❌ 間違い: ローカルパスは使用しない（修正が必要な場合のみ）
.package(path: "../database-kit"),
```

ローカルパスを使用するのは、database-kit 自体に変更・修正が必要な場合のみ。

### storage-kit

**storage-kit は trait の条件付き伝播を使用するため、安定後は GitHub URL に戻すこと。**

```swift
// ✅ 安定後: GitHub URL + traits
.package(url: "https://github.com/1amageek/storage-kit.git", branch: "main",
    traits: [
        .trait(name: "FoundationDB", condition: .when(traits: ["FoundationDB"])),
        .trait(name: "SQLite", condition: .when(traits: ["SQLite"])),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1amageek/database-framework](https://github.com/1amageek/database-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
