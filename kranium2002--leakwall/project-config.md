---
trigger: always_on
description: LeakWall is a single Rust binary that protects developers from AI coding agent security failures. It provides four commands:
---

# LeakWall — AI Agent Security Platform

## Project Overview

LeakWall is a single Rust binary that protects developers from AI coding agent security failures. It provides four commands:
- `leakwall scan` — audit MCP servers, skill files, configs, and cross-reference external registries
- `leakwall run` — runtime HTTPS MITM proxy with live TUI dashboard or headless JSONL logging
- `leakwall watch` — background daemon monitoring filesystem for security-relevant changes
- `leakwall report` — generate HTML/SARIF/JSON reports from scan results

## Build & Test Commands

```bash
cargo build                       # Debug build
cargo test                        # Run all 156 unit tests
cargo clippy -- -D warnings       # Lint (zero warnings enforced)
cargo fmt --check                 # Verify formatting
cargo build --release             # Optimized release binary
cargo install --path crates/leakwall-cli  # Install to ~/.cargo/bin
```

All four checks must pass before committing. Run `cargo fmt` to auto-fix formatting.

## Architecture

Ten crates in a Cargo workspace (`crates/*`):

```
leakwall-cli (binary: "leakwall")
├── leakwall-secrets    — secret discovery, fingerprinting, Aho-Corasick + regex scanning
├── leakwall-mcp        — MCP config discovery, tool poisoning detection, AgentAudit registry queries, CVE matching
├── leakwall-proxy      — MITM TLS proxy (hyper + rustls), CA cert gen, request interception, redaction
│   └── leakwall-secrets
├── leakwall-tui        — ratatui dashboard, event stream, session/scan reports
│   └── leakwall-proxy
├── leakwall-skills     — skill file discovery and static security analysis
├── leakwall-watch      — filesystem monitoring daemon, IPC, desktop notifications
│   ├── leakwall-mcp
│   ├── leakwall-secrets
│   └── leakwall-skills
├── leakwall-behavior   — behavioral anomaly detection, data-shape classification, allowlists
├── leakwall-report     — HTML/SARIF/JSON report generation, trend analysis, cost estimation
└── leakwall-stdio      — stdio interception for MCP servers, JSON-RPC parsing, secret scanning
```

## Key Conventions

### Error Handling
- **Library crates**: `thiserror` enums (`SecretError`, `ProxyError`, `McpError`, `TuiError`, `SkillsError`, `WatchError`, `BehaviorError`, `ReportError`, `StdioError`)
- **CLI binary**: `anyhow::Result` with `.context()` for human-readable messages
- **No `unwrap()` or `expect()` in library code** — only in tests and `main.rs`

### Async
- `tokio` runtime, `#[tokio::main]` only on the binary crate
- `spawn_blocking` for CPU-intensive work (Aho-Corasick scanning, regex matching)
- `broadcast::channel` for proxy → TUI event fan-out and stdio → event fan-out
- `Arc<RwLock<T>>` for shared mutable state, `DashMap` for cert cache
- `oneshot::channel` for proxy bind readiness signaling

### Security
- `subtle::ConstantTimeEq` for proxy auth comparison (prevents timing attacks)
- `fs2::FileExt` for file locking on PID files, hash pins, and cache files
- `semver` for proper semantic version comparison in CVE matching
- `std::sync::OnceLock` for infallible lazy regex compilation (no `expect()`)
- No `unsafe` code anywhere
- CA private key permissions enforced (chmod 600 on Unix)
- IPC message size limit: 1 MB; pipe line size limit: 10 MB
- Symlink traversal prevention: `follow_links(false)` on WalkDir
- Private/RFC1918 address blocking on passthrough tunnels
- Oversized request bodies return 413 instead of forwarding unscanned
- UTF-8 boundary-safe string slicing throughout (use `char_indices()` or `.get()`, never `&s[..n]`)
- Strip `proxy-authorization` header from forwarded requests (prevents auth token leaking upstream)
- No `as u32`/`as u16` narrowing casts — use `u32::try_from(val).unwrap_or(u32::MAX)` to prevent silent truncation
- Percent-encode all URI-special characters (`%` first, then `#`, `?`, space, `\`) in generated URIs

### Correctness
- Async file writes must be flushed after each write for real-time log consumers (`tail -f`)
- IPC command handlers must trigger the actual operation, not just return acknowledgement
- Deletion/removal detection: iterate the stored/old state to find missing items, not just the current state
- Don't run state-comparison logic on empty/default inputs (e.g., skip hash pin check when tool list is empty due to `--no-exec`)
- Guard math functions against degenerate inputs (empty data → early return 0.0, avoid NaN/Inf)
- After case-folding, use the folded string's `.len()` for byte offset calculations, not the original

### Code Style
- `.rustfmt.toml`: max_width=100, field_init_shorthand, try_shorthand
- `#[instrument]` on async functions, `tracing::{debug, info, warn}` for logging
- `bytes::Bytes` in the proxy hot path (zero-copy via Arc)

## Directory Layout

```
crates/
  leakwall-cli/src/        main.rs, cmd_scan.rs, cmd_run.rs, cmd_watch.rs, cmd_report.rs, config.rs
  leakwall-secrets/src/    lib.rs, discovery.rs, patterns.rs, fingerprint.rs, scanner.rs
  leakwall-mcp/src/        lib.rs, discover.rs, connect.rs, analyze.rs, registry.rs, cve.rs, hashpin.rs, cross_origin.rs, command_analysis.rs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kranium2002) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
