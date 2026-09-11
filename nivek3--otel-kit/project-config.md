---
trigger: always_on
description: This document provides context and guidelines for AI agents working with the `tracing-otel-extra` codebase.
---

# tracing-otel-extra

This document provides context and guidelines for AI agents working with the `tracing-otel-extra` codebase.

## Project Overview

**tracing-otel-extra** is a comprehensive Rust library for tracing, logging, and OpenTelemetry integration. It focuses on providing production-ready observability solutions for Axum web applications with minimal boilerplate.

### Key Goals

- Simplify OpenTelemetry setup for Rust applications
- Provide opinionated, production-oriented defaults
- Integrate tracing, metrics, and logging cohesively
- Support the Axum web framework with first-class middleware

## Repository Structure

```
tracing-otel-extra/
├── crates/
│   ├── axum-otel/           # Axum middleware for HTTP tracing
│   ├── tracing-otel/        # Core logging and tracing utilities
│   └── otel-init/           # OpenTelemetry initialization layer
├── examples/
│   ├── otel/                # Basic OpenTelemetry example
│   └── microservices/       # Multi-service distributed tracing demo
├── Cargo.toml               # Workspace configuration
└── docker-compose.yml       # Development infrastructure
```

### Crate Dependencies

```
axum-otel
    └── tracing-otel
            └── otel-init
```

### Crate Boundaries

| Crate | Responsibility |
| ----- | -------------- |
| `otel-init` | OpenTelemetry provider/subscriber bootstrap (`OtelGuard`, OTLP setup) |
| `tracing-otel` | Shared HTTP tracing utilities (`fields`, `context`, `span`, `macros`) and the opinionated `Logger` facade |
| `axum-otel` | Axum/Tower HTTP middleware (`AxumOtelSpanCreator`, `AxumOtelOnResponse`, `AxumOtelOnFailure`) |

- Applications that only need Axum middleware should depend on `axum-otel`.
- Applications that only need provider-level OpenTelemetry setup can use `otel-init`.
- Applications that want the full logging/bootstrap facade should use `tracing-otel` with `logger` or `env`.

Workspace `[workspace.dependencies]` entries must not enable crate features implicitly. Each member crate must declare the `tracing-otel` features its source code actually uses (for example, `axum-otel` enables `context`, `fields`, and `macros`).

## Coding Conventions

### Rust Edition & Toolchain

- **Edition**: Rust 2024 (`edition = "2024"`)
- **Minimum Rust Version**: 1.96.0
- **Resolver**: Cargo resolver v2

### Code Style

1. **Lints**: The codebase uses strict linting (see `axum-otel/src/lib.rs` for reference):
   ```rust
   #![deny(unsafe_code)]
   #![warn(
       missing_docs,
       missing_debug_implementations,
       missing_copy_implementations,
       trivial_casts,
       trivial_numeric_casts,
       unused_import_braces,
       unused_qualifications
   )]
   ```

2. **Documentation**: All public APIs must have doc comments with examples where appropriate.

3. **Error Handling**: Use `anyhow::Result` for application-level errors. Library code should define specific error types when appropriate.

4. **Builder Pattern**: Configuration structs use the builder pattern with `with_*` methods:
   ```rust
   Logger::new("my-service")
       .with_format(LogFormat::Json)
       .with_level(Level::DEBUG)
       .init()
   ```

5. **Imports**: Prefer explicit imports over glob imports. Group imports by:
   - Standard library
   - External crates
   - Internal modules

### Feature Flags

`tracing-otel` has no default features:

| Feature   | Description                      | Depends on |
| --------- | -------------------------------- | ---------- |
| `fields`  | HTTP field extraction helpers    | —          |
| `macros`  | Runtime-configurable `dyn_span!` / `dyn_event!` macros | — |
| `http`    | HTTP context propagation (no tracing bridge) | `fields` |
| `context` | Trace context utilities (`set_otel_parent`, etc.) | `http` |
| `span`    | HTTP span creation utilities     | `context`, `macros` |
| `otel`    | Re-exports `otel-init` | — |
| `logger`  | Opinionated logging/bootstrap facade | `otel` |
| `env`     | Environment-based `Logger` configuration | `logger` |

`axum-otel` uses `context`, `fields`, `macros`; examples use `env`.

## Key Patterns

### 1. Resource Management

OpenTelemetry providers are managed via guard patterns that clean up on drop:

```rust
let _guard = Logger::new("service").init()?;
// Providers are automatically flushed and shut down when _guard is dropped
```

### 2. Tower Integration

The `axum-otel` crate integrates with `tower-http::TraceLayer`:

```rust
TraceLayer::new_for_http()
    .make_span_with(AxumOtelSpanCreator::new().level(Level::INFO))
    .on_response(AxumOtelOnResponse::new())
    .on_failure(AxumOtelOnFailure::new())
```

### 3. OpenTelemetry Context Propagation

The `set_otel_parent` function extracts trace context from HTTP headers:

```rust
pub fn set_otel_parent(headers: &http::HeaderMap, span: &tracing::Span) {
    let remote_context = extract_context_from_headers(headers);
    span.set_parent(remote_context);
    // Record trace_id for logging
}
```

### 4. Dynamic Span Creation

Use the `dyn_span!` macro for runtime-configurable log levels:

```rust
let span = dyn_span!(
    self.level,
    "request",
    http.request.method = %method,
    http.route = route,
    trace_id = Empty
);
```

## Testing

### Running Tests

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nivek3/otel-kit](https://github.com/nivek3/otel-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
