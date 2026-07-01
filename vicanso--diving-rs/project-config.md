---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**diving-rs** is a Rust-based tool for analyzing Docker image layers to understand space usage and inefficiencies. It provides two interfaces:
- **Terminal mode** (default): TUI built with ratatui for interactive layer exploration
- **Web mode**: HTTP server with React frontend for remote analysis

The tool fetches image metadata from registries (Docker Hub, private registries) or local docker/tar files, decompresses layers, and provides detailed file-level visibility into what's consuming space.

## Build System & Common Commands

Build system uses **Cargo** for Rust and **Make** for orchestration.

### Key Make Targets

```bash
# Web frontend
make build-web          # Build React frontend (builds dist/ directory)
make dev-web            # Watch-based dev with cargo-watch

# Rust compilation
make lint               # Run cargo clippy
make fmt                # Format code with cargo fmt
make release            # Build optimized release binary
make dev                # Run terminal mode with test image (redis:alpine)
make dev-docker         # Run with local docker client
make dev-ci             # Run with CI mode enabled

# Pre-commit setup
make hooks              # Install git hooks (runs fmt + lint before commits)
```

### Pre-commit Hook

The repo includes a pre-commit hook (`hooks/pre-commit`) that runs `make fmt && make lint`. Install it with `make hooks`.

## Code Conventions

### Imports: `use` first, no inline fully-qualified paths

When referencing an item from another module, bring it into scope with a `use`
declaration at the top of the file and then call it by its short name. Do **not**
spell out the fully-qualified path inline at the call site.

```rust
// ✅ preferred — import once, use the short name
use crate::i18n::Lang;
use std::collections::HashSet;

fn build(lang: Lang, seen: &HashSet<String>) { /* ... */ }
```

```rust
// ❌ avoid — fully-qualified path inline
fn build(lang: crate::i18n::Lang, seen: &std::collections::HashSet<String>) { /* ... */ }
```

This applies to `crate::` / `super::` paths and to `std`/external crates alike.
Rare exceptions: disambiguating a genuine name collision, or a one-off reference
where a `use` would be misleading — keep those local and obvious.

## Architecture

### Core Design Pattern

The application follows a **dual-mode runtime**: single binary, dual execution paths determined by the `--mode` argument (terminal or web).

### Data Flow: Image Analysis

1. **Image Source Parsing** (`src/main.rs` + `src/image/docker.rs`)
   - Parse image references: `redis:alpine`, `docker://`, `file://`, registry URLs
   - Determine source type: registry (default), docker client, or local tar file

2. **Manifest Resolution** (`src/image/oci_image.rs` + `src/image/docker.rs`)
   - Fetch manifest from Docker V2 registry API (HTTPS)
   - Handle multi-arch images: select by architecture (`?arch=amd64` query param)
   - Support both Docker Schema 2 and OCI Image Index formats

3. **Layer Decompression & Caching** (`src/image/layer.rs` + `src/store/blob.rs`)
   - Download compressed layer blobs from registry
   - Cache locally in `~/.diving/layers/` with TTL (default 90 days)
   - Decompress with gzip/zstd on-demand
   - Metadata stored in blob files

4. **File Tree Construction** (`src/image/oci_image.rs`)
   - Extract file metadata from layer tar archives
   - Build hierarchical file tree with operations (added, modified, deleted)
   - Calculate efficiency score: `(1 - wastedSize / totalSize) * 100`

5. **Output Rendering**
   - **Terminal**: TUI interface via `src/ui/` modules (layers, files, details)
   - **Web**: JSON API via `src/controller.rs` → React frontend in `web/src/`

### Module Structure

- **`src/main.rs`**: Entry point; CLI parsing via clap; mode selection (terminal/web)
- **`src/image/`**: Core image analysis logic
  - `docker.rs`: Registry API client, manifest parsing, image info extraction
  - `layer.rs`: Tar extraction, file parsing from compressed blobs
  - `oci_image.rs`: OCI spec parsing, file tree building, operation tracking
- **`src/ui/`**: Terminal UI (ratatui-based)
  - `mod.rs`: Main app event loop, state management
  - `layers.rs`, `files.rs`, `layer_detail.rs`: Widget definitions
- **`src/controller.rs`: Web API endpoints**
  - `/api/analyze`: Initiate image analysis (returns JSON)
  - `/api/file`: Download individual files from layers
  - `/api/latest-images`: Return recent analyses
  - Fallback handler serves static assets from embedded `dist/`
- **`src/store/blob.rs`**: Async blob file I/O and TTL-based cleanup (runs hourly)
- **`src/config/`**: Configuration loading from `~/.diving/config.yml`
- **`src/error.rs`**: HTTP error types with snafu error handling
- **`src/middleware.rs`**: Axum middleware (access logging, trace ID injection)
- **`src/task_local/`**: Task-local storage for request tracing
- **`src/dist.rs`**: Embedded static asset serving with ETag caching

### Web Frontend (`web/`)

- **Framework**: React 19 + TypeScript + Vite + Ant Design
- **Entry point**: `web/src/main.tsx` → `web/src/App.tsx`
- **Key Components**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vicanso/diving-rs](https://github.com/vicanso/diving-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
