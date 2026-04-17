---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Check Commands

```bash
cargo check          # type-check without building
cargo build          # debug build
cargo build --release # release build (strip + LTO)
cargo test           # run all tests
cargo test <name>    # run a single test by name
```

Requires **nightly** Rust (pinned in `rust-toolchain.toml`).

## Architecture

This is a Rust library crate that provides an opinionated framework for building MCP (Model Context Protocol) servers on top of [`rmcp`](https://crates.io/crates/rmcp). It handles transport selection, authentication, and CLI argument parsing so consumers only need to implement `rmcp::ServerHandler`.

### Entry point pattern

Consumers create a `McpApp` with a name, auth provider, and server factory closure, then call `run()`:

```rust
mcp_framework::run(McpApp {
    name: "my-server",
    auth: AuthProvider::Basic(BasicAuthConfig::from_env().unwrap()),
    server_factory: |token_store, session_store| MyServer::new(token_store, session_store),
    stdio_token_env: Some("MY_TOKEN"),
    session_store: None,
    ..
}).await
```

`run()` (`src/runner.rs`) handles `.env` loading, CLI parsing (clap), tracing setup, and dispatches to the chosen transport.

### Transport layer (`src/transport/`)

Two modes selected via `--transport` CLI flag:
- **HTTP** (`http.rs`): Axum router with `rmcp::StreamableHttpService` at `/mcp`, OAuth well-known endpoints, CORS. `build_app()` is extracted as a pure function for testability.
- **Stdio** (`stdio.rs`): stdin/stdout via `rmcp` transport, used for Claude Desktop local integration.

### Auth layer (`src/auth/`)

`AuthProvider` enum drives which middleware and routes are registered:
- **None**: no auth middleware
- **Basic**: HTTP Basic auth middleware, credentials from `BASIC_AUTH_*` env vars
- **OAuth**: Full OAuth2/OIDC proxy for Keycloak — includes RFC 8414/9728 metadata endpoints, RFC 7591 dynamic client registration, PKCE authorization flow, and token proxying. All OAuth routes live under `/oauth/`.

Key type: `TokenStore` — thread-safe token storage shared between auth middleware and the server handler via the factory closure. Supports automatic token refresh for OAuth mode.

### Session layer (`src/session/`)

`SessionStore<T>` — generic, thread-safe per-session data store with TTL expiration. The type parameter `T` (must implement `Send + Sync + Default + Clone + 'static`) is defined by the consumer. Default TTL is 30 minutes. A background cleanup task purges expired sessions in HTTP mode.

Helper function `resolve_session_id(extensions)` extracts the `mcp-session-id` header from MCP request context extensions, falling back to `"default"` for stdio mode.

### Audit logging (`src/audit/`)

Pluggable tool call audit logging. Every `call_tool` invocation can be logged via a `ToolCallLogger` trait implementation. The framework ships two built-in loggers:
- `NoopLogger` — discards all records
- `TracingLogger` — emits structured `tracing::info!` events

Key types:
- `ToolCallRecord` — captures tool name, arguments (`Option<Map<String, Value>>`), session ID, timestamp (`SystemTime`), duration (`Duration`), dispatch source (registry vs inner handler), and outcome
- `ToolCallOutcome` — `Success { is_error, content_summary }` or `McpError { code, message }`. `is_error: true` means the tool reported a tool-level error (e.g. bad LLM input) but the MCP protocol call itself succeeded
- `ToolCallSource` — `Registry` (dynamic tools from `CapabilityRegistry`) or `Inner` (static tools from `ServerHandler`)

Logging is fire-and-forget via `tokio::spawn` — zero impact on tool call latency. When no logger is configured, the hot path has zero overhead (no clones, no allocations).

The interception point is `DynamicHandler::call_tool` in `src/capability/handler.rs`.

#### Using a built-in logger

```rust
McpAppBuilder::new("my-server")
    .tool_call_logger(Arc::new(TracingLogger))
    .server(|| MyServer::new())
    .run()
    .await?;
```

#### Implementing a custom storage backend

Implement the `ToolCallLogger` trait. The `log` method returns `Pin<Box<dyn Future<Output = ()> + Send>>` — this allows async I/O (database writes, HTTP calls). Handle errors internally; the framework cannot act on them since logging is fire-and-forget.

```rust
use mcp_framework::audit::{ToolCallLogger, ToolCallRecord, ToolCallOutcome};
use std::future::Future;
use std::pin::Pin;

struct FileLogger { path: std::path::PathBuf }

impl ToolCallLogger for FileLogger {
    fn log(&self, record: ToolCallRecord) -> Pin<Box<dyn Future<Output = ()> + Send>> {
        let path = self.path.clone();
        Box::pin(async move {
            let line = format!(
                "{} tool={} session={} duration={}ms outcome={}\n",
                humantime::format_rfc3339(record.timestamp),
                record.tool_name,
                record.session_id,
                record.duration.as_millis(),
                match &record.outcome {
                    ToolCallOutcome::Success { is_error, .. } =>
                        if *is_error { "tool_error" } else { "success" },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Plawn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
