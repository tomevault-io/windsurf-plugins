---
trigger: always_on
description: This file provides guidance to Claude Code when working on this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working on this repository.

## Project Overview

EchoFS is a single-binary file server written in Rust. It serves a local directory over HTTP with a browser-based UI for directory browsing, media preview, and file sharing. It also provides read-only WebDAV access, allowing file managers (macOS Finder, Windows Explorer, Linux Nautilus) to mount and browse the served directory as a network drive.

## Build & Run

```bash
# Check compilation
cargo check

# Build release binary (~1.4 MB)
cargo build --release

# Run (serves current directory on port 8080)
./target/release/echofs

# Run with options
./target/release/echofs --root /path/to/dir --port 9000 --open

# Log to file instead of stdout
./target/release/echofs --log /var/log/echofs.log

# Disable access logging
./target/release/echofs --log off

# Show hidden files and directories
./target/release/echofs --show-hidden

# Limit directory browsing depth
./target/release/echofs --max-depth 1

# Only allow browsing root directory (no subdirectory access)
./target/release/echofs -d 0

# Limit download speed per request to 1MB/s
./target/release/echofs --speed-limit 1m

# Speed limit with suffix: 500k (KB/s), 2m (MB/s), 1g (GB/s)
./target/release/echofs -s 500k

# WebDAV is enabled by default (read-write); disable it with:
./target/release/echofs --no-webdav

# Require authentication for WebDAV access (does not affect browser/web UI)
./target/release/echofs --webdav-user admin --webdav-pass secret

# Mount via macOS Finder: Go → Connect to Server → http://localhost:8080
# Mount via Windows Explorer: Map Network Drive → \\localhost@8080\
```

## Architecture

Single-binary SPA architecture: the HTML/CSS/JS is embedded in `template.rs` and served inline. The frontend fetches directory data from JSON API endpoints and renders client-side.

### Source Files

- `lib.rs` — Library crate root: re-exports all modules as `pub mod` for use by `main.rs` and integration tests
- `main.rs` — Entry point: CLI parsing, LAN IP detection, server startup; imports modules from the `echofs` library crate
- `cli.rs` — clap derive CLI arguments (root, port, bind, open, log, show-hidden, max-depth, speed-limit, no-webdav, webdav-user, webdav-pass)
- `server.rs` — Axum router setup, CORS layer, access log middleware, TCP listener; conditionally registers WebDAV routes (PROPFIND, OPTIONS) via `any()` handlers
- `handlers.rs` — Route handlers: serves HTML for directories, streams files with Range support, JSON API; errors are dispatched via `AppError::into_response_for(&headers)` to return HTML for browsers or JSON for AJAX
- `range.rs` — HTTP Range header parsing, builds 200/206/416 responses with streaming body; supports optional per-request speed limiting via `ThrottledRead` wrapper
- `directory.rs` — Async directory listing with path traversal protection (`canonicalize` + `starts_with`), conditional hidden file access blocking (controlled by `--show-hidden` flag), and directory depth limiting (controlled by `--max-depth` flag); all filesystem I/O runs in `tokio::task::spawn_blocking` to avoid blocking the async runtime
- `template.rs` — Embedded SPA (HTML/CSS/JS) with dark/light theme, responsive layout, media preview modal, dynamic page title; also provides `error_html()` for styled error pages
- `mime_utils.rs` — MIME detection via `mime_guess`, file type icon mapping
- `error.rs` — `AppError` enum with dual-mode responses: `into_response_for(headers)` returns HTML error pages for browser requests and JSON for AJAX requests; also implements `IntoResponse` (JSON-only) as fallback
- `logging.rs` — Access log axum middleware; `LogTarget` enum (Stdout/Off/File) drives output; uses `ConnectInfo<SocketAddr>` for client IP and `tokio::sync::Mutex` for file writes
- `throttle.rs` — `ThrottledRead<R: AsyncRead>` wrapper that limits read throughput using a token-bucket algorithm; also provides `parse_speed()` for human-readable rate strings (e.g. `500k`, `1m`)
- `webdav.rs` — Full read-write WebDAV support: handles PROPFIND (Depth 0/1), OPTIONS, LOCK, UNLOCK (read methods) and PUT, DELETE, MKCOL, COPY, MOVE, PROPPATCH (write methods); generates `207 Multi-Status` XML responses (`DAV:multistatus`) with resource properties (`displayname`, `getcontentlength`, `getlastmodified`, `getcontenttype`, `resourcetype`); provides `check_auth()` for Basic Auth enforcement via `--webdav-user`/`--webdav-pass` CLI flags (protects all operations when configured); reuses `directory::safe_resolve()` and `directory::safe_resolve_parent()` for path safety; XML is built via `XmlWriter` helper with no external XML library; enabled by default, disabled with `--no-webdav`

### Tests

- `src/*.rs` — Each source module contains `#[cfg(test)] mod tests` with unit tests (48 total)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dengsgo/echofs](https://github.com/dengsgo/echofs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
