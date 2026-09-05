---
trigger: always_on
description: This file provides guidance for AI coding agents working in the
---

# AGENTS.md

This file provides guidance for AI coding agents working in the
`infra-controller` repository.

## Project Overview

**NVIDIA Infra Controller (NICo)** is an API-based microservice written in Rust
and Golang that provides site-local, zero-trust, bare-metal lifecycle
management with DPU-enforced isolation. It automates the complexity of the
bare-metal lifecycle to fast-track building next-generation AI Cloud offerings.

> **Status:** Active development. APIs, configurations, and features may
> change without notice between releases.

### Key Responsibilities

- Hardware inventory management and orchestration
- Redfish-based hardware management
- Hardware testing and firmware updates
- IP address allocation and DNS services
- Power control (on/off/reset)
- Provisioning, wiping, and node-release orchestration
- Machine trust enforcement during tenant switching

## Repository Structure

```text
infra-controller/
├── crates/              # Rust crate implementations. To discover all crates
│                        # and their purpose, run `ls crates/` or see the
│                        # [workspace] members list in `Cargo.toml` — each
│                        # crate's own `Cargo.toml` has a `description` field.
│                        # Note: the directory name does NOT always equal the
│                        # crate name (e.g. crates/api/ → crate nico-api).
│                        # Use `grep '^name =' crates/<dir>/Cargo.toml | head -1`
│                        # to get the actual crate name before running
│                        # `cargo test -p <name>` or similar.
├── book/                # mdBook documentation
├── deploy/              # Kubernetes deployment configs and Kustomization overlays
├── dev/                 # Local dev tools (Dockerfiles, test configs, certs)
├── helm/                # Helm chart for Kubernetes deployment
├── bluefield/           # BlueField DPU-specific components
├── pxe/                 # PXE boot artifact generation
├── lints/               # Custom Clippy lints (carbide-lints crate)
├── include/             # Shared Makefile fragments
├── .github/             # GitHub Actions workflows and templates
├── rest-api/            # Golang-based REST API
├── Cargo.toml           # Workspace dependency management
├── Makefile.toml        # Primary build/task automation
├── Makefile-build.toml  # Build-specific tasks
└── Makefile-package.toml # Packaging tasks
```

## Technology Stack

### gRPC API and components

- **Language:** Rust (edition 2024, toolchain pinned in `rust-toolchain.toml`)
- **Async runtime:** Tokio
- **gRPC framework:** Tonic (with TLS via Rustls/aws_lc_rs)
- **HTTP framework:** Axum (pinned; see `Cargo.toml` for compatibility rationale)
- **Database:** SQLx (compile-time checked queries)
- **Observability:** OpenTelemetry, Tracing (structured logfmt logging)
- **Build tool:** `cargo-make` (TOML task runner)
- **API definitions:** Protocol Buffers (protobuf)

### REST API and components

- **Language (REST API):** Golang 1.26.x

## Build, Test, and Lint Commands

### REST API contract conventions

- Do not use `omitempty` on REST API response fields. Clients must be able to
  distinguish an empty value from a field unsupported by the API version.
- Paginated operations must implement deterministic ordering before pagination
  and document every supported `orderBy` value and its default in OpenAPI. Do
  not rely on an upstream API or database's implicit result order.

All task automation uses `cargo-make`. Install it with:

```bash
cargo install cargo-make
```

### Building

```bash
# Standard debug build (all workspace crates)
cargo build

# Release build
cargo build --release

# Full CI build + test (mirrors what CI runs)
cargo make build-and-test-release-container-services

# Build the admin CLI locally
cargo make build-cli
```

### Testing

```bash
# Run all tests
cargo test

# Build prerequisites first, then test (recommended for integration tests)
cargo make correctly-execute-tests
```

When writing tests, prefer the **table-driven** style and helpers from
`carbide-test-support`; use the [Testing section in `STYLE_GUIDE.md`](STYLE_GUIDE.md#testing)
for table structure and API details. Use grouped `scenarios!` / `value_scenarios!`
or explicit `check_cases` / `check_values` when cases share one operation and
assertion form. When cases share setup but require different assertions, use a
local case table that keeps each case's check next to its inputs.

Before adding coverage, inventory the relevant unit, database, controller, and
integration tests. Each new test should have one reason to exist: an observable
contract or distinct failure boundary that no retained test protects. Use the
smallest set of cases that exercise different behavior. Do not enumerate a
Cartesian product merely because inputs are booleans or enums; enumerate a
combination only when it is reachable and protects distinct observable behavior
or a distinct failure boundary, including precedence between conflicting inputs.

Place each proof at the narrowest layer that can exercise the contract.
Higher-level tests should prove wiring, persistence, transaction behavior,
concurrency, or external effects that lower-level tests cannot; do not repeat a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dsx-ai-factory/infra-controller](https://github.com/dsx-ai-factory/infra-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
