---
trigger: always_on
description: Follow these rules when working on E2EE (end-to-end encryption) features, crypto store, device key management, or any file touching matrix-sdk encryption
---


# Rust Matrix E2EE Development Rules

## Project Context

This is `matrix-mcp-server-r2`, a Rust MCP (Model Context Protocol) server providing Matrix homeserver tools to AI agents. The v2 milestone adds E2EE (end-to-end encryption) support so agents can operate in encrypted Matrix rooms.

### Crate Versions (pin these in all code and suggestions)

- `matrix-sdk` = 0.10 (with feature `e2e-encryption` via the `e2ee` Cargo feature flag)
- `ruma` = 0.12 (with features `client-api-c`, `compat`)
- `rmcp` = 1.2 (MCP SDK)
- `tokio` = 1.43 (async runtime)
- `axum` = 0.8 (HTTP framework)
- `thiserror` = 2.0 (error types)
- Rust edition = 2021, MSRV = 1.88

### Feature Flag

E2EE is gated behind a Cargo feature flag:
```toml
[features]
e2ee = ["matrix-sdk/e2e-encryption"]
```
All E2EE code paths must be conditionally compiled with `#[cfg(feature = "e2ee")]` so the server can still build and run without encryption support.

## Encryption Architecture

### How matrix-sdk E2EE Works

When the `e2e-encryption` feature is enabled, `matrix-sdk` handles encryption **transparently**:

- `room.send(content)` automatically encrypts if the room has `m.room.encryption` state
- Incoming encrypted events are automatically decrypted during sync
- The `OlmMachine` state machine (from `matrix-sdk-crypto`) manages all Olm/Megolm sessions
- `vodozemac` (pure Rust, replaces deprecated `libolm`) provides the underlying Olm/Megolm crypto primitives

### What We Must Implement

1. **Persistent crypto store** -- `SqliteCryptoStore` for device keys and Megolm sessions that survives container restarts
2. **Device key bootstrap** -- publish identity keys and one-time prekeys on first run
3. **Cross-signing bootstrap** -- automated (no human interaction) during agent startup
4. **Encrypted room creation** -- set `m.room.encryption` state event
5. **Decryption failure handling** -- surface "Unable to decrypt" errors gracefully in MCP tool responses
6. **Encryption status exposure** -- tools to query room/device encryption state

## Rust Coding Patterns

### Ownership and Borrowing

- Prefer borrowing (`&self`, `&str`, `&[T]`) over cloning
- When `.clone()` is needed for async moves (e.g., spawning tasks), comment why
- Use `Arc<T>` for shared state across async tasks; avoid `Arc<Mutex<T>>` when message passing via channels is viable
- The `MatrixMcpServer` struct already uses `Arc<Config>` -- follow this pattern for any shared crypto state

### Async Patterns

- All async code runs on Tokio 1.x multi-threaded runtime
- Use `tokio::sync::Mutex` (not `std::sync::Mutex`) when a lock must be held across `.await` points
- The OlmMachine requires locking for `outgoing_requests()`, `get_missing_sessions()`, and `share_room_key()` to prevent duplicate requests -- use a dedicated `tokio::sync::Mutex<()>` guard per the matrix-sdk-crypto tutorial

### Error Handling

- Use `thiserror` for error type definitions in `src/error.rs`
- Add E2EE-specific error variants to `AppError`:
  ```rust
  #[error("E2EE not enabled (build with --features e2ee)")]
  E2eeNotEnabled,
  #[error("Crypto store error: {0}")]
  CryptoStore(String),
  #[error("Decryption failed: {0}")]
  DecryptionFailed(String),
  #[error("Device key bootstrap failed: {0}")]
  KeyBootstrapFailed(String),
  ```
- In MCP tool handlers, return `CallToolResult::error(...)` for decryption failures rather than panicking
- Distinguish between "room is encrypted but we can't decrypt" vs "room is not encrypted" in responses

### Conditional Compilation

Gate all E2EE code behind the feature flag:
```rust
#[cfg(feature = "e2ee")]
mod e2ee {
    // E2EE-specific modules
}

#[cfg(feature = "e2ee")]
pub async fn bootstrap_e2ee(&self) -> Result<CallToolResult, McpError> {
    // ...
}

#[cfg(not(feature = "e2ee"))]
pub async fn bootstrap_e2ee(&self) -> Result<CallToolResult, McpError> {
    Ok(err_text("E2EE support not enabled. Rebuild with --features e2ee"))
}
```

## Matrix SDK E2EE Patterns

### Client Builder with Crypto Store

When E2EE is enabled, the client must use a persistent store:
```rust
#[cfg(feature = "e2ee")]
let client = Client::builder()
    .homeserver_url(homeserver_url)
    .sqlite_store("/data/crypto-store", None)  // passphrase optional
    .build()
    .await?;
```

### Creating Encrypted Rooms

Set the `m.room.encryption` state event during room creation:
```rust
use ruma::events::room::encryption::RoomEncryptionEventContent;
use ruma::events::room::history_visibility::{HistoryVisibility, RoomHistoryVisibilityEventContent};

let mut req = CreateRoomRequest::new();
req.initial_state = vec![
    InitialStateEvent::new(RoomEncryptionEventContent::with_recommended_defaults())
        .to_raw_any(),
    InitialStateEvent::new(RoomHistoryVisibilityEventContent::new(HistoryVisibility::Joined))
        .to_raw_any(),
];
```

### Handling Decryption Failures

When parsing timeline events, check for encrypted events that couldn't be decrypted:
```rust
if ev.get("type").and_then(|t| t.as_str()) == Some("m.room.encrypted") {
    // Event could not be decrypted -- include a clear indicator in the response
    let algorithm = ev.pointer("/content/algorithm")
        .and_then(|a| a.as_str())

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/l0r3zz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
