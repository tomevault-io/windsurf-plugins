---
trigger: always_on
description: An all-in-one PHP application server written in Rust that embeds PHP via FFI into a single binary. Runs WordPress, Laravel, etc. without external PHP-FPM. Includes embedded SQLite (via litewire), gossip clustering, and a built-in KV store.
---

# ePHPm — Embedded PHP Manager

An all-in-one PHP application server written in Rust that embeds PHP via FFI into a single binary. Runs WordPress, Laravel, etc. without external PHP-FPM. Includes embedded SQLite (via litewire), gossip clustering, and a built-in KV store.

## Build & Run

```bash
# Stub mode (no PHP, fast iteration on HTTP/routing logic)
cargo build

# Release binary with PHP + sqld embedded
cargo xtask release           # → target/release/ephpm (PHP 8.5, sqld auto-downloaded)
cargo xtask release 8.4       # → target/release/ephpm (PHP 8.4)
cargo xtask release --no-sqld # skip sqld embedding (single-node SQLite only)

# Windows .exe (cross-compiled from WSL, requires cargo-xwin)
cargo install cargo-xwin
cargo xtask release --target windows       # → target/x86_64-pc-windows-msvc/release/ephpm.exe
# Note: Windows builds never include sqld (no Windows binary available from Turso)
```

Prerequisites for `cargo xtask release`: git, curl, tar, `build-essential`, `pkg-config`, `libclang-dev` (for bindgen), and `musl-tools`/`musl-dev` on Linux (the prebuilt `libphp.a` is musl-linked). The xtask downloads the PHP SDK from `github.com/ephpm/php-sdk` releases and the sqld binary from Turso releases — no PHP CLI, Composer, or static-php-cli needed.

The PHP SDK is cached at `php-sdk/<version>-<os>-<arch>/` (e.g. `php-sdk/8.5.2-linux-x86_64/`). Delete that directory to force a re-download.

## Testing

```bash
cargo test -p <crate> <test_name>          # run a single test (preferred)
cargo test -p <crate>                      # run all tests in a crate
cargo test --workspace                     # all tests (may fail without openssl for e2e deps)
cargo clippy --workspace --all-targets -- -D warnings  # lint (pedantic, warnings = errors)
cargo +nightly fmt --all -- --check        # format check (nightly required for import grouping)
cargo deny check                           # license/advisory audit
```

IMPORTANT: Run single tests when possible, not the full suite. Use `cargo test -p <crate> <test_name>`. `cargo nextest` is preferred but may not be installed — fall back to `cargo test`.

The `ephpm-e2e` crate is **excluded from the workspace** — it runs inside Docker via `cargo xtask e2e` and has different dependencies. Don't try to compile it with `cargo test --workspace`.

## Workspace Structure

| Crate | Purpose |
|-------|---------|
| `ephpm` | CLI binary — clap args, config loading, server startup, graceful shutdown |
| `ephpm-server` | HTTP server (hyper + tokio) — routing, static files, TLS/ACME, metrics, litewire/SQLite startup, query stats |
| `ephpm-php` | PHP embedding via FFI — SAPI implementation, worker thread pool, request/response mapping |
| `ephpm-config` | Configuration (figment) — TOML + env var overrides (`EPHPM_` prefix) |
| `ephpm-kv` | Embedded KV store — DashMap, RESP2 protocol, TTL/expiry, compression (gzip/zstd/brotli) |
| `ephpm-db` | DB proxy — MySQL wire protocol, connection pooling, R/W splitting |
| `ephpm-cluster` | Clustering — SWIM gossip (chitchat), consistent hash ring, KV replication, SQLite primary election |
| `ephpm-sqld` | sqld embedding — binary extraction via `include_bytes!()`, child process lifecycle, health checks |
| `ephpm-query-stats` | Query observability — SQL normalization, digest tracking, slow query logging, Prometheus metrics |
| `xtask` | Build & test tooling — `release`, `php-sdk`, `e2e`, `e2e-up`, `e2e-down` |

## External Dependencies

| Dependency | Location | Purpose |
|-----------|----------|---------|
| **litewire** | `../litewire/crates/litewire` (path dep) | MySQL/Hrana wire protocol → SQLite translation proxy |
| **PHP SDK** | Downloaded by `cargo xtask php-sdk` from `github.com/ephpm/php-sdk` releases | Prebuilt `libphp.a` (Linux/macOS) or `php8embed.{dll,lib}` (Windows) plus PHP headers. Pinned per minor in `xtask/src/main.rs::PHP_SDK_VERSIONS` |
| **sqld** | Embedded via `include_bytes!()` at build time | SQLite replication server for clustered mode (v0.24.32 pinned in xtask) |

litewire is a standalone project at `github.com/ephpm/litewire`. It's used as a library — ePHPm calls `LiteWire::new(backend).mysql(addr).serve()`.

The PHP SDK is built by a separate pipeline at `github.com/ephpm/php-sdk` (uses static-php-cli internally). ePHPm itself doesn't depend on static-php-cli at all — it just consumes the resulting tarballs.

## Architecture: Database

Three database modes, all transparent to PHP (`pdo_mysql` connects to `127.0.0.1:3306`):

1. **DB Proxy** (`[db.mysql]`) — forwards MySQL wire traffic to a real MySQL/PG server with connection pooling
2. **Single-node SQLite** (`[db.sqlite]`) — litewire + rusqlite in-process, no external database
3. **Clustered SQLite** (`[db.sqlite]` + `[cluster]`) — litewire + sqld sidecar, WAL frame replication via gRPC

Mode detection (`is_clustered_sqlite()` in `crates/ephpm-server/src/lib.rs`):
- If `replication.role = "primary"` or `"replica"` → clustered
- If `replication.role = "auto"` AND `cluster.enabled = true` → clustered (election via gossip)
- Otherwise → single-node (rusqlite in-process)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ephpm/ephpm](https://github.com/ephpm/ephpm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
