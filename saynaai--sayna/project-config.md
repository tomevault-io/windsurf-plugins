---
trigger: always_on
description: Comprehensive best practices for Axum 0.8+ development with WebSocket support
---

# Axum Best Practices: A Comprehensive Guide (0.8+)

This guide provides best practices for developing applications using the Axum framework in Rust, enhanced with WebSocket support for building atomic web services. It covers code organization, common patterns, performance, security, WebSocket specifics, testing, pitfalls, and tooling.

**Note:** This project uses **Axum 0.8.x** (released January 2025). Minimum Rust version: 1.75.

## 1. Code Organization and Structure

A clear project layout improves maintainability and scalability. Consider the following structure:

```
project_root/
├── src/
│   ├── main.rs             # Application entry point (sets up Router + server)
│   ├── lib.rs              # Shared library for core logic
│   ├── routes/             # Route definitions
│   │   ├── mod.rs
│   │   ├── api.rs          # REST endpoints
│   │   ├── ws.rs           # WebSocket routes and upgrade handlers
│   ├── handlers/           # Request handlers and WebSocket message handlers
│   │   ├── mod.rs
│   │   ├── api_handlers.rs
│   │   ├── ws_handlers.rs
│   ├── services/           # Business logic services
│   │   ├── mod.rs
│   │   ├── user_service.rs
│   │   ├── chat_service.rs # WebSocket message routing
│   ├── middleware/         # Tower middleware layers
│   │   ├── mod.rs
│   │   ├── auth.rs
│   │   ├── logging.rs
│   ├── state/              # Shared application state
│   │   ├── mod.rs          # Defines AppState struct (use `State<AppState>` extractor)
│   ├── errors/             # Error types and conversion to responses
│   │   ├── mod.rs
│   │   ├── app_error.rs    # Implements `IntoResponse`
│   ├── utils/              # Utility functions (e.g., DB pool setup)
│   │   ├── mod.rs
│   │   ├── db.rs
├── tests/                  # Integration tests
│   ├── api_tests.rs        # Tests for HTTP endpoints
│   └── ws_tests.rs         # WebSocket integration tests
├── Dockerfile              # Containerization
├── Cargo.toml
└── .env
```

### 1.1 Naming Conventions

* **Modules & Files:** lowercase with underscores (e.g., `ws_handlers.rs`, `user_service.rs`).
* **Structs & Enums:** CamelCase (e.g., `AppState`, `ChatMessage`).
* **Functions:** snake\_case reflecting action (e.g., `create_user`, `handle_ws_message`).

### 1.2 Modular Boundaries

* **Router vs. Handler:** Keep route declarations (`Router::route`) in `routes/` and logic in `handlers/` or `services/`.
* **State Management:** Use `State<AppState>` for compile-time checked shared state (DB pools, config, caches). Reserve `Extension` only for per-request data injected by middleware (e.g., authenticated user info).
* **Separation of Concerns:** Handlers focus on HTTP/WebSocket framing; services manage business logic.

### 1.3 State vs Extension (Important)

* **Prefer `State<T>`** over `Extension<T>` for application-wide state:
  - `State` is type-checked at compile time; `Extension` gives runtime errors if type mismatches
  - `State` is faster than `Extension`
  - Use `.with_state(app_state)` when building the router
* **Use `Extension<T>`** only for:
  - Per-request data set by middleware (e.g., `Extension<AuthenticatedUser>`)
  - Dynamic data that varies per request

## 2. Common Patterns and Anti-patterns

### 2.1 Axum-Specific Patterns

* **Extractors:** Use built‑in extractors (`Path`, `Query`, `Json`, `State`) for typed data parsing.
* **Tower Middleware:** Leverage `tower-http` layers for CORS, compression, and tracing.
* **Error Handling:** Define custom error types implementing `IntoResponse` to standardize responses.
* **State Management:** Use `State<AppState>` extractor. Wrap shared resources in `Arc` within AppState and use `tokio::sync` types (e.g., `RwLock`, `broadcast`) for interior mutability.

### 2.2 Anti-patterns to Avoid

* **Blocking in Handlers:** Never perform blocking I/O; use `spawn_blocking` or async drivers.
* **Global Mutable State:** Avoid static mutable variables; prefer injected state.
* **Large Handlers:** Break complex flows into smaller service functions.
* **Ignoring WebSocket Backpressure:** Always handle send/receive delays to prevent buffer exhaustion.

## 3. Performance Considerations

* **Asynchronous Ecosystem:** Use async database clients (`sqlx`, `tokio-postgres`).
* **Connection Pooling:** Configure DB pools for optimal concurrency.
* **HTTP/2 & Keep-Alive:** Enable keep-alive and HTTP/2 via Hyper configuration in `Server::builder`.
* **Compression & Caching:** Add `CompressionLayer` and `Cache` middlewares from `tower-http`.
* **Efficient Serialization:** Use `serde` with `#[serde(crate = "serde")]` and consider `bincode` or `MessagePack` for WS when binary.

## 4. Security Best Practices

* **CORS:** Permit only trusted origins with `CorsLayer`.
* **TLS:** Terminate TLS at reverse proxy or configure Hyper with `rustls`.
* **Input Validation:** Rely on extractor types and manual checks for complex data.
* **Authentication/Authorization:** Use JWT or session-based auth; protect WS upgrades by validating tokens in the handshake.
* **Rate Limiting:** Apply `RateLimitLayer` to APIs and WS endpoints.

## 5. WebSocket Best Practices

* **Upgrade Endpoint:** Define a route using `get(ws_handler)` and call `WebSocketUpgrade::on_upgrade`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SaynaAI/sayna](https://github.com/SaynaAI/sayna) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
