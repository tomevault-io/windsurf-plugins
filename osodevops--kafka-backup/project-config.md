---
trigger: always_on
description: This document provides context for AI-assisted development on this project.
---

# CLAUDE.md - Development Guide for OSO Kafka Backup & Restore

This document provides context for AI-assisted development on this project.

## Project Overview

**OSO Kafka Backup & Restore** is a high-performance, production-grade tool written in Rust for backing up and restoring Apache Kafka topics to cloud storage or local filesystem. It supports point-in-time recovery (PITR) with millisecond precision and handles consumer group offset recovery across different clusters.

### Key Problems Solved
- Durable backup of Kafka topics to S3, Azure Blob, GCS, or local filesystem
- Disaster recovery with exact data fidelity
- Point-in-time restore (PITR) with millisecond precision
- Consumer group offset recovery across different clusters (offset space discontinuity problem)
- Deployment-agnostic: works on bare metal, VM, Docker, Kubernetes

## Repository Structure

```
kafka-backup/
├── Cargo.toml                    # Workspace root
├── crates/
│   ├── kafka-backup-core/        # Core library (no K8s dependencies)
│   │   └── src/
│   │       ├── lib.rs            # Module exports
│   │       ├── backup/           # Backup engine
│   │       ├── restore/          # Restore engine + offset handling
│   │       ├── kafka/            # Kafka protocol client
│   │       ├── storage/          # Multi-backend storage abstraction
│   │       ├── segment/          # Record batching & serialization
│   │       ├── offset_store/     # SQLite-based offset tracking
│   │       ├── config.rs         # Configuration structures
│   │       ├── manifest.rs       # Backup manifest & reports
│   │       ├── compression.rs    # Compression algorithms
│   │       ├── error.rs          # Error types
│   │       ├── circuit_breaker.rs
│   │       ├── health.rs
│   │       └── metrics/          # Prometheus metrics
│   └── kafka-backup-cli/         # Binary CLI wrapper
│       └── src/
│           ├── main.rs           # CLI entry point
│           └── commands/         # Command implementations
├── config/                       # Config templates
└── docs/                         # Comprehensive documentation
```

## Build & Test Commands

```bash
# Build the project
cargo build

# Build release binary
cargo build --release

# Run tests
cargo test

# Run tests with output
cargo test -- --nocapture

# Check compilation without building
cargo check

# Format code
cargo fmt

# Run clippy lints
cargo clippy

# Run the CLI
cargo run -p kafka-backup-cli -- --help
cargo run -p kafka-backup-cli -- backup --config config/backup.yaml
cargo run -p kafka-backup-cli -- restore --config config/restore.yaml
```

## CI Pre-commit Checklist

**You MUST run these checks before every commit. CI will reject PRs that fail any of them.**

```bash
# 1. Format code (CI runs: cargo fmt --all -- --check)
cargo fmt --all

# 2. Clippy with CI-identical flags (CI runs with -D warnings)
cargo clippy --all-targets --all-features -- -D warnings

# 3. Run tests
cargo test
```

### Version Bumping Rules

Any PR that changes files under `crates/`, `Cargo.toml`, `Cargo.lock`, or `Dockerfile` **must** bump `[workspace.package].version` in the root `Cargo.toml`. CI enforces this via `scripts/ci/check-release-version.py`.

**Semver for 0.x crates (current):**
- **Patch bump** (0.12.1 → 0.12.2): bug fixes, internal changes, no public API changes
- **Minor bump** (0.12.x → 0.13.0): any breaking change to `kafka-backup-core` public API

**What counts as a breaking change** (detected by `cargo-semver-checks`):
- Adding a public field to a public struct (breaks struct literal construction)
- Removing or renaming public fields, methods, or types
- Changing the type of a public field or method signature
- Adding required parameters to public functions

**After bumping the version:**
```bash
# Update Cargo.lock to match
cargo check
# Then stage both Cargo.toml and Cargo.lock
```

### Semver-safe patterns

To add data to public structs without a breaking change:
- Use `#[non_exhaustive]` on the struct (but adding it is itself a one-time break)
- Add methods instead of public fields
- Use builder patterns for construction

## Architecture Overview

```
┌──────────────────────────────────────────────┐
│  CLI Layer (kafka-backup-cli)                │
│  - backup, restore, list, describe, validate │
│  - offset-reset, three-phase orchestration   │
└────────────────┬─────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────┐
│  Core Engine (kafka-backup-core library)     │
├──────────────────────────────────────────────┤
│  ├─ Backup Engine                            │
│  │  ├─ Segment Writer (mini-batching)        │
│  │  ├─ Compression (zstd/lz4)                │
│  │  └─ Manifest Management                   │
│  ├─ Restore Engine                           │
│  │  ├─ Segment Reader & Decompression        │
│  │  ├─ PITR Filtering (time-window)          │
│  │  └─ Partition Remapping                   │
│  ├─ Offset Management                        │
│  │  ├─ SQLite-based Offset Store             │
│  │  ├─ Consumer Group Reset Executor         │
│  │  └─ Three-Phase Restore Orchestrator      │
│  ├─ Kafka Client                             │
│  │  ├─ kafka-protocol crate (raw protocol)   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [osodevops/kafka-backup](https://github.com/osodevops/kafka-backup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
