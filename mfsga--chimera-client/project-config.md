---
trigger: always_on
description: - You are an agentic contributor living inside the Chimera Client workspace. This file is your quick reference for how to build, lint, test, and style the code before handing it back.
---

# AGENTS.md

## Why you open this file
- You are an agentic contributor living inside the Chimera Client workspace. This file is your quick reference for how to build, lint, test, and style the code before handing it back.
- Treat every change as part of a workspace that mirrors Clash but lives in Rust; keep things fast, observable, and safe.
- When in doubt, read this file first, then consult a crate README or the `docs/` folder for deeper context.

## Workspace at a glance
- **Rust workspace** defined in `Cargo.toml` with pinned toolchain `stable` (see `rust-toolchain.toml`; a `nightly-2025-09-15` channel is also listed but commented out).
- **Crates:**
  - `clash-bin` (`clash-rs` CLI entrypoint with `clap` parsing and `start_scaffold`).
  - `clash-lib` core runtime scaffolding; contains `app/`, `common/`, `config/`, `proxy/`, and `session/` modules.
  - `clash-dns`, `clash-ffi`, `clash-doc`, `clash-netstack` are empty placeholders but keep the namespace reserved.
  - `docs/` holds human-facing explanations; `config.yaml` is the sample profile generated on first run.
- **Scripting helpers:** `start.sh`/`start.ps1` wrap `cargo watch` for quick iteration, `make` recipes live at the repo root.

## Toolchain & dependencies
- **Rust:** Use the channel specified in `rust-toolchain.toml` (currently `stable`).
- **Required native tools:** CMake ≥3.29, LLVM/libclang, nasm (Windows), protoc for proto generation; install via your platform package manager.
- **Notifications:** `cargo fmt`, `cargo clippy --all-targets --all-features`, and `cargo test --all` are treated as basic hygiene before any PR.

## Build/Lint/Test commands

### Build
- `cargo build` – builds the entire workspace in debug mode (default). Run this when you need the latest artifacts.
- `cargo build --release` – builds optimized artifacts; only run if you need release performance.
- `cargo check --all` – fastest full-workspace sanity check; recommended before `cargo fmt` when you just changed logic.

### Format & lint
- `cargo fmt` – canonical formatting; never skip before pushing.
- `cargo clippy --all-targets --all-features` – catches lints across libs and binaries. Use `-- -D warnings` only when lint clean-up is on your TODO list.

### Tests
- `cargo test --all` – runs all tests across every crate.
- `CLASH_RS_CI=true cargo test --all --all-features` – CI-like invocation that enables extra guards.
- `make test-no-docker` – wraps the same tests without Docker; helpful if you rely on local tooling.
- `cargo test -p clash-lib` (or another crate name) – narrow the scope to a single crate.

### Running a single test (or subset)
- Use `cargo test -p <crate> <substring>` to run focused tests. Example: `cargo test -p clash-lib parser_can_load_config`.
- Add `-- --exact` when you need an exact test name; append `--nocapture` to see `tracing` logs.
- When you need specific features, combine `-p` with `--features shadowsocks,tuic` and `-- --test-threads=1` for deterministic output.

### Running the client
- `cargo run -p clash-rs -- -c config.yaml` – mimics the user experience: loads `config.yaml`, sets up logging, starts the async runtime.
- `./start.sh` (or `start.ps1` on Windows) – wraps `cargo watch -x "run -p clash-rs -- -c config.yaml"` for live reload.
- Pass `-t` to test config, `--help-improve` to toggle telemetry, `--directory` to point to a profile folder, or `--config` for alternate files.

### External Controller API

The API server is configured via `external-controller` + `secret` in the YAML config:

| Field | Example | Notes |
|---|---|---|
| `external-controller` | `127.0.0.1:13456` | REST + WebSocket API address |
| `secret` | `chimera` | Auth token |
| `cors-allow-origins` | `["http://localhost:3000"]` | CORS origins; default `Any` if absent |

**Auth mechanisms:**
- **HTTP REST**: `Authorization: Bearer <secret>` header
- **WebSocket**: `?token=<secret>` query parameter

**Key API paths:**
- `/configs`, `/proxies`, `/rules`, `/group`, `/version`, `/logs`, `/traffic`, `/memory`, `/connections`, `/dns`, `/flows`
- WebSocket variants live under `/ws/*` (`/ws/memory`, `/ws/connections`, `/ws/traffic`, `/ws/logs`, `/ws/flows`)
- The `websocket_uri_rewrite` middleware auto-redirects WS upgrade requests from `/memory` → `/ws/memory` (etc.), so clients connecting to the plain path work transparently.

### Documentation
- `make docs` – rebuilds documentation for `docs/` pages and other static assets.
- `cargo doc -p clash_doc --no-deps` – regenerates API docs for the config reference generator.
- `cargo doc` – when you need to inspect generated docs for any crate.

## Code style & contribution guidance

### Imports & grouping
- Group imports by origin: first `std`, then external crates (`tokio`, `tracing`, `serde`, etc.), then workspace crates (`clash_lib::app`).
- Keep groups sorted alphabetically within braces for readability. Example in `clash-lib/src/lib.rs`: `use std::{collections::HashMap, ...};`.
- Avoid wildcard imports unless you are re-exporting a whole module; prefer explicit items for clarity.

### Formatting & spacing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MFSGA/Chimera_Client](https://github.com/MFSGA/Chimera_Client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
