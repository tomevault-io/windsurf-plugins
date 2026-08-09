---
trigger: always_on
description: SWS is a cross-platform, high-performance, async static file server written in Rust. Built on `hyper` v1 + `tokio`. Binary target: ~4MB. Edition 2024, MSRV 1.88.0.
---

# AGENTS.md — Static Web Server (SWS)

SWS is a cross-platform, high-performance, async static file server written in Rust. Built on `hyper` v1 + `tokio`. Binary target: ~4MB. Edition 2024, MSRV 1.88.0.

## Quick Start

```bash
# Build with all features
cargo build --release --features all

# Lint (must pass with zero warnings before commit)
cargo clippy --features all -- -D warnings
cargo clippy --features all --tests -- -D warnings

# Format check
cargo fmt --all -- --check

# Run tests
cargo test -v --features all
cargo test -v --no-default-features
```

## Project Rules (non-negotiable)

- `#![forbid(unsafe_code)]` — no `unsafe` anywhere
- No `unwrap()` / `expect()` in production code (use `?` or match)
- No commented-out code — delete it, git preserves history
- All clippy warnings treated as errors
- New dependencies must not increase binary size >100KB unless they replace existing functionality or gate behind a feature flag

## Architecture

```
settings/  →  server/opts.rs  →  handler.rs  →  static_files.rs
 (parse)        (init)             (pipeline)      (serve file)
```

### Request Pipeline (linear, 3 phases)

**Pre-processing** (may short-circuit): method check → health/metrics → CORS → basic auth → maintenance → redirects → rewrites → virtual hosts → markdown negotiation

**Core**: `static_files::handle()` — path resolution, index files, dir listing, pre-compressed variants, byte-range

**Post-processing** (additive, runs on every response): fallback page → CORS headers → text charset → static compression → dynamic compression → cache-control → security headers → custom headers

Rules: pre-processing steps return early when they handle the request. Post-processing steps are additive — no step removes headers set by prior steps. Custom headers take final precedence.

### Key Modules

| Module | Purpose |
|--------|---------|
| `settings/` | Parse CLI (clap) / env / TOML, merge, validate |
| `server/` | Bind listener, HTTP/1 or HTTP/2+TLS, graceful shutdown |
| `handler.rs` | Orchestrate request pipeline |
| `static_files.rs` | Path resolution, index files, dir listing, byte-range, pre-compressed |
| `compression.rs` | On-the-fly gzip/deflate/brotli/zstd |
| `compression_static.rs` | Serve pre-compressed `.br`/`.gz`/`.zst` files |
| `security_headers.rs` | HSTS, CSP, X-Frame-Options, etc. |
| `control_headers.rs` | Cache-Control based on file extension |
| `cors.rs` | CORS preflight and header injection |
| `fs/` | File system: path sanitization, metadata, streaming |
| `directory_listing/` | HTML/JSON directory index |
| `basic_auth.rs` | BCrypt-based HTTP basic auth |
| `error.rs` | Crate error types (`Error`, `Result<T>`) |
| `body.rs` | Unified response body (`BoxBody`) |

## Configuration

Three equivalent channels, one `General` struct. Precedence (highest→lowest): CLI args → env vars → TOML config → compiled defaults. Feature-gated settings use `#[cfg(feature = "...")]`. Validate everything at startup — never per-request.

## Feature Flags (Cargo + `#[cfg]`)

| Feature | Module | Description |
|---------|--------|-------------|
| `compression` | `compression.rs` + `compression_static.rs` | On-the-fly + pre-compressed serving |
| `directory-listing` | `directory_listing/` | HTML/JSON directory index |
| `directory-listing-download` | (depends on `directory-listing`) | Tar.gz directory download |
| `http2` | (requires `tls`) | HTTP/2 via `hyper-util/http2` |
| `tls` | `tls.rs` | TLS plumbing (requires a crypto provider) |
| `tls-ring` | (default crypto) | TLS via `ring` |
| `tls-fips` | FIPS TLS | TLS via `aws-lc-rs` |
| `basic-auth` | `basic_auth.rs` | BCrypt HTTP basic auth |
| `fallback-page` | `fallback_page.rs` | Custom 404 page |
| `metrics` | `metrics.rs` | Prometheus metrics endpoint |
| `mem-cache` | `mem_cache/` | In-memory file cache (`mini-moka`) |
| `experimental` | `metrics.rs` + `mem_cache/` | Requires `RUSTFLAGS="--cfg tokio_unstable"` |

Default features: `compression`, `http2`, `tls-ring`, `directory-listing`, `directory-listing-download`, `basic-auth`, `fallback-page`, `metrics`, `mem-cache`.

## Coding Conventions

### Error Handling
- Crate types: `crate::Result<T>` and `crate::Error` (anyhow). Use `StatusCode` for HTTP-level errors
- Wrap with context: `fallible_op().with_context(|| "failed to parse config")?`
- Log at boundaries: modules return errors, `handler.rs` logs and converts to status codes

### Async
- Runtime: `tokio` (multi-threaded). HTTP framework: `hyper` v1
- Offload CPU-heavy work via `tokio::task::spawn_blocking`
- Never call `block_on` inside async context

### File System
- Canonicalize root dir once at startup (`server/opts.rs`)
- Path traversal: `sanitize_path()` → `resolve_and_contain()` → symlink policy check → hidden file check
- Prefer `&Path` over `&PathBuf`, `impl AsRef<Path>` for public APIs

### Response
- Use `crate::body::empty()`, `crate::body::full(x)`, or `crate::body::stream(s)`
- Stream files — never buffer full file in memory (exception: small generated responses)

### Performance
- Pre-compute at startup: canonicalize paths, compile regex, build automata

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [static-web-server/static-web-server](https://github.com/static-web-server/static-web-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
