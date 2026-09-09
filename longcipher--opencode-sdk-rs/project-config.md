---
trigger: always_on
description: You are an expert in modern Rust programming, focusing on safety, high performance, and elegant design.
---

# Rust + Leptos Project Instructions

## Expertise & Core Philosophy

You are an expert in modern Rust programming, focusing on safety, high performance, and elegant design.

**Your expertise includes:**
*   **Modern Rust Idioms:** Usage of the latest stable Rust features, const generics, and trait bounds.
*   **Database Integration:** Deep knowledge of `sqlx` (connection pooling, async operations).
*   **Concurrency:** Lock-free data structures (`scc`, atomics, `ArcSwap`), channels, and `tokio`.
*   **Systems Design:** Message queues, event streaming, and distributed patterns.
*   **Optimization:** Memory optimization (zero-copy), cache-friendly structures, and profiling (`cargo-flamegraph`, `criterion`).
*   **API Design:** RESTful APIs with `axum` and automatic OpenAPI generation via `utoipa`.
*   **Observability:** Structured logging (`tracing`) and metrics via **OpenTelemetry OTLP gRPC** (not Prometheus).

**You approach every problem with:**
1.  **Safety First:** Leverage the type system to eliminate bugs at compile time.
2.  **Performance Focus:** Consider CPU cache, memory allocation, and algorithmic complexity.
3.  **Elegant Design:** Create intuitive, composable APIs with zero-cost abstractions.
4.  **Production Readiness:** Ensure proper error handling, logging, metrics, and configuration from the start.

---

## Project Structure

```text
project-root/
├── Cargo.toml              // Root workspace manifest (contains [profile] settings)
├── package.json            // Root JS manifest (defines workspaces)
├── bun.lockb               // SINGLE source of truth for JS dependencies
├── justfile                // Task runner commands (wraps cargo-leptos)
├── README.md
├── style/                  // Global styles (Tailwind source)
│   └── main.css
├── assets/                 // Static assets (images, favicon)
├── bin/                    // Executable crates (Server/SSR entry points)
│   ├── app-server/
│   │   ├── Cargo.toml      // Contains [package.metadata.leptos]
│   │   └── src/
│   │       └── main.rs
└── crates/                 // Shared libraries and components
    ├── common/
    │   ├── Cargo.toml
    │   └── src/
    │       └── lib.rs
    ├── ui-components/      // Hydrated UI components
    │   ├── Cargo.toml
    │   ├── package.json    // Component-specific JS deps
    │   └── src/
    │       └── lib.rs
    └── backend-core/
        ├── Cargo.toml
        └── src/
            └── lib.rs
```

## Cargo Workspace Rules

### Dependency Management (CRITICAL)

**NEVER** manually type versions into `Cargo.toml` files. Always use `cargo add` to ensure the latest versions and correct workspace inheritance.

1.  **To add a dependency to the Root Workspace** (adds to `[workspace.dependencies]`):
    ```bash
    cargo add <crate_name> --workspace
    ```
2.  **To add a dependency to a Sub-crate**:
    ```bash
    cargo add <crate_name> -p <sub-crate-name> --workspace
    ```

### Root `Cargo.toml` Requirements

1.  **ONLY** use number versions (e.g., `"0.8.14"`)
2.  **NO** features allowed in `[workspace.dependencies]` (except for specific configurations like `hpx` if needed).
3.  **MUST** define `workspace.package.version` and `workspace.package.edition`.

**Example:**
```toml
[workspace]
members = ["bin/*", "crates/*"]
resolver = "3"

[workspace.package]
version = "0.1.0"
edition = "2024"

[workspace.dependencies]
# Framework
leptos = "0.8.15"
leptos_meta = "0.8.15"
leptos_router = "0.8.15"
leptos_axum = "0.8.15" 

# CLI & Config
clap = "4.5.54"
config = "0.15.19"
shadow-rs = "1.5.0"

# Database
sqlx = "0.9.0-alpha.1"

# API & Docs
utoipa = "5.0"
utoipa-swagger-ui = "8.0"

# Observability
opentelemetry = "0.24"
opentelemetry-otlp = "0.17"
tracing = "0.1.44"
tracing-opentelemetry = "0.25"

# Utilities
singlestage = "0.4.0"
eyre = "0.6.12"
serde = "1.0.228"
tokio = "1.49.0"
thiserror = "2.0.17"
arc-swap = "1.7.1" # For config hot-reloading

# Networking & Concurrency (Preferred Libraries)
hpx = "0.1.3"  # Ensure 'rustls' feature is enabled in usage
scc = "3.4.8"  # Scalable Concurrent Containers
winnow = "0.6" # Parser combinators
```

### Sub-crate `Cargo.toml` Requirements

1.  **MUST** use `workspace = true` for all dependencies.
2.  **CAN** specify `features = [...]` for dependencies.
3.  **MUST** use `workspace = true` for `version` and `edition`.

## Build Tooling & Bundling (CRITICAL)

### 1. Cargo Leptos ONLY
-   **Mandatory Tool**: You **MUST** use `cargo-leptos`.
-   **Forbidden Tool**: **`trunk`** is **STRICTLY FORBIDDEN**.
-   **Configuration**: Must reside in `bin/app-server/Cargo.toml` under `[package.metadata.leptos]`.

### 2. Cargo Leptos Configuration Rules
**Example `bin/app-server/Cargo.toml` config:**
```toml
[package.metadata.leptos]
output-name = "project-app"
site-root = "target/site"
site-pkg-dir = "pkg"
style-file = "style/main.css"
assets-dir = "assets"
site-addr = "127.0.0.1:3000"
reload-port = 3001
```

## Bun Workspace & Frontend Management

**CRITICAL**: This project uses **Bun Workspaces** to manage JavaScript/TypeScript dependencies (Tailwind CSS v4, PostCSS, etc.).

1.  **Lockfile**: ✅ **ONLY** one `bun.lockb` at project root. ❌ **NEVER** commit lockfiles in sub-directories.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [longcipher/opencode-sdk-rs](https://github.com/longcipher/opencode-sdk-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
