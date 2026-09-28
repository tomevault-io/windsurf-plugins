---
trigger: always_on
description: enables cache fields by date, and never inherits support from
---

# AGENTS.md — serial-mcp

## Fast truth

- Root server: `src/main.rs` selects stdio vs HTTP transport, parses CLI limits (`--profiles-path`, `--capture-dir` + capture quotas included), and mounts HTTP at `/mcp`.
- MCP surface lives in `src/server.rs`; tool handlers are split under `src/tools/`, prompts under `src/prompts/`, resources under `src/resources/`.
- MCP version policy (`src/mcp_protocol.rs`): the product-owned `SUPPORTED_PROTOCOLS` table is the SINGLE source for advertised versions, lifecycle admission, capability views, and cache shaping. Exactly two rows — `2026-07-28` (preferred, `DiscoverStateless`, `ImmediatePrivate` cache, subscriptions on) and permanent `2025-11-25` (`InitializeSession`, `Omit` cache, subscriptions off). Lookup is exact-match only (`policy_for`); `cache_fields_for(Option<ProtocolVersion>)` grants fields ONLY to rows with `CachePolicy::ImmediatePrivate` — never by date/range, and unknown/future versions inherit no policy. `supported_protocol_versions()` / preferred policy derive from the table, never from `ProtocolVersion::KNOWN_VERSIONS`.
- Dual MCP lifecycle (`src/server.rs`): preferred modern `2026-07-28` discovery/stateless requests (`server/discover` + self-contained per-request `_meta`) and compatible legacy `2025-11-25` initialize/session requests. `get_info()` serves the MODERN view (rmcp intersects `subscriptions/listen` filters against `get_info().capabilities`), `initialize()` returns the legacy view with subscription disabled, `discover()` the modern view with `resources.subscribe: true`. Common surface: tools/resources/prompts/completions (no logging, list-change, or tasks). `accepted_subscription_filter`/`listen` are backed by one process-wide `ResourceEventHub` (`src/resource_events.rs`, capacity 256) shared by every stdio/HTTP handler and the port hotplug watcher; legacy `subscriptions/listen` stays `-32601`.
- `SerialHandler` is built via `SerialHandler::builder()...build()` (`src/server.rs`); the old `with_manager*` telescoping constructors are gone and `with_profiles()` is gone. The builder defaults `profile_store` to an ephemeral store and `capture_store` to disabled. `SerialHandler::new()` tries the OS default profile store and falls back to an ephemeral store with a warning. Production `main.rs` injects the resolved `profile_store`, a `CaptureStore` built from `--capture-dir` + quota flags (disabled by default), and `SystemPortProvider` through the builder.
- Profiles live in a process-wide `Arc<ProfileStore>` (`src/profile_store.rs`), shared by every stdio/HTTP session handler. `main.rs` resolves the path (`--profiles-path` or the OS user-config default, failing startup on an unavailable config dir or invalid file) and injects one store. Persistent mutations take a process-local async mutex, then `spawn_blocking` + an advisory lock on `<file>.lock`, reload-under-lock, `NamedTempFile` + `sync_all` + rename; the cache (shared `Arc<RwLock>`) is published from inside the blocking transaction right after the durable write, before the lock is released — so a cancelled awaiting tool still converges (cache never changes on failed write). `update_defaults_preserving_selector` returns the effective profile atomically (no racy second lookup). File format is schema-versioned TOML (v1 legacy auto-migrates in memory; `schema_version == 0` or `> 2` rejects startup). `Profile` carries `metadata` (revision/timestamps/generated/use_count) and a bounded `revisions` history (max 5 prior snapshots) for the profile-session feature.
- Shared RX framing lives in `src/tools/rx_consume.rs` (`consume_frames` + `RxFrameSink` trait + `disconnect_state`); `read` routes framing through it, but its raw (no-framing) path stays per-tool by design (see "Invariants easy to break").
- Connection lifecycle is in `src/serial/`; shared RX/TX coordination is in `src/rx_session.rs` (always-on pump + ring buffer), `src/tx_session.rs`, and `src/stop_controller.rs`. The pump appends all received bytes to `src/rx_ring.rs`; `read` reads from the ring via cursors. `ConnectionManager`'s connection-opening boundary is an injectable `ConnectionOpener` (`with_opener`): production uses `SystemConnectionOpener` (`SerialConnection::open`); tests inject in-memory backends (`tests/common/controlled.rs::ControlledConnectionOpener`) so the public MCP surface runs cross-platform without an OS serial port. Linux production-path PTY tests use real PTYs; macOS and Windows use normal Rust and controlled-backend coverage.
- Low-level shared primitives: `src/util.rs` (`find_subsequence`, the byte-substring search imported directly by `framing` and the matcher) and `src/precedence.rs` (`resolve_field`, the four-layer framing/parser/protocol precedence helper shared by `io_ops`). Both `pub(crate)`.
- `build.rs` injects `GIT_HASH` / `GIT_HASH_AVAILABLE` / `BUILD_TARGET`.

## Commands worth using

```bash
cargo fmt --all -- --check
cargo build --all-targets --locked
cargo test --locked
cargo clippy --all-targets --locked -- -D warnings

# focused runs
cargo test --lib <test_name>
cargo test --test <file_stem> <test_name>
cargo test --test serial_pty
cargo test --test http_integration
cargo test --test stdio_integration
cargo test --test config_schema_validation

# networked schema drift check

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qarnet/serial-mcp](https://github.com/qarnet/serial-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
