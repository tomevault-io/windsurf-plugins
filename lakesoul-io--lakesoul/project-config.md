---
trigger: always_on
description: LakeSoul is a cloud-native Lakehouse framework (LF AI & Data sandbox project, v3.0.0).
---

# Agent Guidelines for LakeSoul

## Project Overview

LakeSoul is a cloud-native Lakehouse framework (LF AI & Data sandbox project, v3.0.0).
It provides ACID transactions, LSM-Tree style upserts, schema evolution, CDC ingestion,
time travel, and unified batch/streaming processing.

The codebase is split across two build systems:
- **Rust** (Cargo workspace) — native metadata client and IO/merge layer
- **Java/Scala** (Maven multi-module) — Spark, Flink, and Presto integrations, plus a Java JNI bridge to the Rust layer

---

## Repository Layout

```
LakeSoul/
├── rust/                        # Cargo workspace root (active members below)
│   ├── proto/                   # Protobuf definitions (entity.proto) + generated Rust types
│   ├── lakesoul-metadata/       # PostgreSQL metadata client (tokio-postgres, bb8 pool)
│   ├── lakesoul-metadata-c/     # C FFI shim over lakesoul-metadata (for Java JNI)
│   ├── lakesoul-io/             # Native IO: Arrow/DataFusion/Parquet read-merge-write
│   ├── lakesoul-io-c/           # C FFI shim over lakesoul-io (for Java JNI)
│   ├── lakesoul-datafusion/     # DataFusion SQL integration 
│   ├── lakesoul-flight/         # Arrow Flight server (workspace-disabled, WIP)
│   ├── lakesoul-s3-proxy/       # S3 proxy (workspace-disabled, WIP)
│   └── lakesoul-console/        # CLI console (workspace-disabled, WIP)
│
├── native-io/
│   └── lakesoul-io-java/        # Java JNI bridge (loads liblakesoul_io_c.so / .dylib)
│
├── lakesoul-common/             # Shared Java utilities (protobuf, config, etc.)
├── lakesoul-spark/              # Apache Spark 3.3 integration (Scala 2.12)
├── lakesoul-flink/              # Apache Flink integration (Java + Scala)
├── lakesoul-presto/             # Presto/Trino integration
├── lakesoul-spark-gluten/       # Spark + Gluten/Velox integration (optional Maven profile)
│
├── python/                      # Python bindings (maturin/PyO3, pyproject.toml)
│
├── script/
│   ├── meta_init.sql            # PostgreSQL schema DDL
│   ├── meta_init_for_local_test.sh
│   ├── meta_rbac_init.sql       # RBAC row-level security policies
│   └── meta_cleanup.sql
│
└── docker/                      # Docker-compose setups for local dev
```

---

## Tech Stack

| Component | Technology |
|---|---|
| Metadata store | PostgreSQL 14+ (ACID, MVCC, RBAC row-level security) |
| Native IO/merge | Rust — DataFusion, Arrow, Parquet, object_store|
| Async runtime | Tokio (full features) |
| gRPC | Tonic |
| Java bridge | JNI via `lakesoul-io-c` / `lakesoul-metadata-c` (C FFI) |
| Spark | Apache Spark 3.5.8, Scala 2.12 |
| Flink | Apache Flink (Java + Scala) |
| Python | PyO3 + maturin, pyarrow |
| Build (Rust) | Cargo stable toolchain + rustfmt + clippy |
| Build (JVM) | Maven, Java 8 target |

---

## Build Commands

### Prerequisites

1. **PostgreSQL 14+** running locally with a test database:
   ```sh
   ./script/meta_init_for_local_test.sh -j 1
   ```
   Default test credentials: user=`lakesoul_test`, password=`lakesoul_test`, db=`lakesoul_test`

2. **protoc** (Protocol Buffers compiler) — required for Rust `build.rs` code generation.

3. **JDK 11** (for Maven builds, despite Java 8 source/target compatibility).

---

### Rust

```sh
# Build all active workspace members
cargo -q build

# Build release
cargo -q build --release

# Run all tests (requires PostgreSQL)
RUST_BACKTRACE=full cargo test

# Run tests for a specific package
cargo -q test --package lakesoul-io
cargo -q test --package lakesoul-metadata

# Lint
cargo fmt --all --check
cargo clippy

# Build the C FFI libraries (used by Java JNI)
cargo -q build --release -p lakesoul-io-c
cargo -q build --release -p lakesoul-metadata-c
# Output: rust/target/release/liblakesoul_io_c.so (Linux) or .dylib (macOS)
```

The Rust toolchain is pinned to `stable` in `rust-toolchain.toml`.

---

### Java / Maven (Spark & Flink)

The Java build **requires** the Rust C FFI `.so`/`.dylib` files to be built first
and placed at `rust/target/release/`.

```sh
# Build everything (skip tests)
mvn -q -B clean package -DskipTests --file pom.xml

# Run Spark tests (subset 1)
mvn -q -B clean test -pl lakesoul-spark -am -Pcross-build -Pparallel-test \
    -Dtest='UpdateScalaSuite,ReadSuite,...' -Dsurefire.failIfNoSpecifiedTests=false

# Run Flink tests
mvn -q -B clean test -pl lakesoul-flink -am -Pcross-build

# Enable Gluten/Velox profile
mvn -q -B clean package -Pgluten -DskipTests

# Generate test report
mvn surefire-report:report-only -pl lakesoul-spark -am
```

---

### Python

The Python package uses [maturin](https://github.com/PyO3/maturin) to compile the
Rust extension and [uv](https://github.com/astral-sh/uv) for dependency management.

```sh
cd python

# Install dev dependencies
uv sync --group dev

# Build and install the Rust extension in-place (development mode)
uv run maturin develop

# Run tests
uv run pytest tests/

# Build a release wheel
uv run maturin build --release
```

---

## Key Architecture Concepts

### Metadata Layer (`rust/lakesoul-metadata`)
- All table/partition/data-commit metadata is stored in **PostgreSQL**.
- The `DaoType` enum in `src/lib.rs` enumerates every SQL operation (select, insert, update, delete).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lakesoul-io/LakeSoul](https://github.com/lakesoul-io/LakeSoul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
