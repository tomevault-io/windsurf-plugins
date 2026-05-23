---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Drasi Server repository - a standalone server wrapper around DrasiLib that provides REST API, configuration management, and server lifecycle features for Microsoft's Drasi data processing system. The actual core functionality is provided by the external drasi-lib library located at `../drasi-lib/`.

## Development Commands

### Build and Run
- Build: `cargo build`
- Build release: `cargo build --release`
- Cross-compile: `make build-cross TARGET=x86_64-pc-windows-gnu`
- Run server: `cargo run` or `cargo run -- --config config/server.yaml`
- Run with custom port: `cargo run -- --port 8080`
- Run with plugin verification disabled: `cargo run -- --skip-verification --config config/server.yaml`
- Run with UI disabled: `cargo run -- --disable-ui`
- Run with UI enabled (override config): `cargo run -- --enable-ui`
- Validate config (structure only): `cargo run -- validate --config config/server.yaml`
- Validate config (with plugins): `cargo run -- validate --config config/server.yaml --plugins-dir ./plugins`
- Check compilation: `cargo check`

### Plugin Loading
Plugins (sources, reactions, bootstrap providers) are loaded at runtime as cdylib shared libraries (`.so`/`.dylib`/`.dll`) from a `plugins/` directory next to the binary. Each plugin is self-contained with its own tokio runtime, communicating via a stable C ABI. Plugin building is managed by drasi-core, not this repository.

**Important: `[patch.crates-io]` does NOT affect plugins.** Cargo patches only affect compile-time dependency resolution for the server binary. Plugins are separate shared libraries loaded at runtime — they must be built separately. When developing with local drasi-core changes, always use `make build-local-plugins` to rebuild plugins from local source. Registry-downloaded plugins (`autoInstallPlugins: true`) will NOT be ABI-compatible with local drasi-core changes.

- Build all plugins from local drasi-core (release): `make build-local-plugins`
- Build all plugins from local drasi-core (debug): `make build-local-plugins-debug`
- Build test-only plugins (mock, log, scriptfile): `make build-local-test-plugins`
- Download test plugins from OCI registry (no drasi-core needed): `make download-test-plugins`

**Local directory plugin sources:** The `pluginRegistry` config field (and `--registry` CLI flag) accepts local filesystem paths in addition to OCI registry URLs. When a path is detected (e.g., `/path/to/plugins`, `./plugins`, `../drasi-core/target/debug/plugins`, `file:///opt/plugins`), the system scans the directory for plugin binaries instead of contacting an OCI registry. This is useful for development workflows where plugins are built locally. Detection is cross-platform: Unix absolute paths, relative paths (`./`, `../`), home-relative (`~/`), `file://` URIs, Windows drive letters, and UNC paths are all recognized as local directories.

### Testing
- Run all tests: `cargo test`
- Run all tests including plugin-dependent: `make test-all`
- Run unit tests only: `cargo test --lib`
- Run specific test: `cargo test test_name`
- Run integration tests: `./tests/run_working_tests.sh`
- Run plugin smoke tests: `make test-smoke`
- Run with logging: `RUST_LOG=debug cargo test -- --nocapture`
- Run host-sdk integration tests: `cd ../drasi-core && cargo test -p drasi-host-sdk --test integration_test`

### Code Quality
- Format code: `cargo fmt`
- Run linter: `cargo clippy`
- Fix linter warnings: `cargo clippy --fix`

## Architecture

### DrasiServer Components (This Repository)

This repository contains only the server wrapper functionality:

1. **Server** (`src/server.rs`) - Main server implementation that wraps DrasiLib
2. **API** (`src/api/`) - REST API implementation with OpenAPI documentation
   - `v1/` - API version 1 handlers, routes, and OpenAPI spec
   - `v1/plugin_handlers.rs` - Plugin management API endpoints
   - `shared/` - Common handlers, error types, and response types shared across versions
   - `version.rs` - API version constants and utilities
   - `models/` - Data Transfer Objects (DTOs)
   - `mappings/` - DTO to domain model conversions
3. **Builder** (`src/builder.rs`) - Builder pattern for server construction
4. **Main** (`src/main.rs`) - CLI entry point for standalone server
5. **Dynamic Loading** (`src/dynamic_loading.rs`) - Runtime plugin loading from shared libraries
6. **Plugin Operations** (`src/plugin_operations.rs`) - Shared plugin management service used by CLI, init, startup, and API
7. **Plugin Orchestrator** (`src/plugin_orchestrator.rs`) - Server-level plugin lifecycle coordination (load, install, track)
8. **Plugin Registry** (`src/plugin_registry.rs`) - Re-exports from host-sdk: mutable registry with `Arc<RwLock<PluginRegistry>>`

### Core Components (External Dependency)

The actual data processing functionality is provided by drasi-lib:

1. **Sources** - Data ingestion from various systems (PostgreSQL, HTTP, gRPC, etc.)
2. **Queries** - Continuous Cypher queries over data with joins and bootstrap
3. **Reactions** - Automated responses to changes (webhooks, SSE, logging, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drasi-project/drasi-server](https://github.com/drasi-project/drasi-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
