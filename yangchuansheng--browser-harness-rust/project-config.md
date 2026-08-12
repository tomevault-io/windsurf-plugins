---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Browser Harness Rust Upstream Sync**

This project is a Rust-native reimplementation of `browser-use/browser-harness`. The current work is a brownfield sync effort: replicate all applicable upstream behavior from `browser-use/browser-harness` commits after April 21, 2026 into the Rust runtime without forcing the Python repository layout onto this codebase.

The audience is agents and developers who need the Browser Harness thesis as a durable Rust runtime: typed CLIs, daemon/control-plane crates, reusable domain knowledge, and optional WASM guest workflows.

**Core Value:** The Rust implementation must preserve behavior parity with upstream Browser Harness updates while remaining idiomatic, typed, and maintainable in the existing Rust architecture.

### Constraints

- **Architecture**: Preserve the Rust workspace and crate boundaries — the migration must adapt upstream behavior into existing Rust crates and binaries.
- **Traceability**: Maintain a migration audit with upstream commit references and applicability decisions.
- **Safety**: Do not expose secrets from upstream docs or local environment; scan generated docs and changed files before commit.
- **Verification**: Run `cargo fmt --check`, `cargo test --workspace`, and targeted CLI smoke checks where possible.
- **Network dependency**: Upstream commit analysis depends on the fetched `upstream/main` remote.
- **Recording**: For recording or video tasks, follow `SKILL.md` and
  `interaction-skills/make-video.md`. A natural request to show, record, or demo
  the work opts in; significant work alone does not. Keep the exact path returned
  by `start_recording()` and never reenact a finished task. The Rust recording
  engine is not yet ported.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- Rust 2021 edition - active runtime, CLI, daemon, protocol, CDP client, WASM host, and smoke runner live in `rust/Cargo.toml`, `rust/bins/*/src/main.rs`, and `rust/crates/*/src/lib.rs`.
- Rust package version `0.1.0` - inherited from `[workspace.package]` in `rust/Cargo.toml` by workspace members such as `rust/crates/bh-protocol/Cargo.toml` and `rust/bins/browser-harness-cli/Cargo.toml`.
- WebAssembly Text (WAT) - minimal guest samples are stored in `rust/guests/navigate_and_read.wat`, `rust/guests/persistent_counter.wat`, and `rust/guests/persistent_browser_state.wat`.
- Rust-to-WASM guest crates - packaged guest workflows live under `rust/guests/rust-*/Cargo.toml` and compile as `cdylib` for `wasm32-unknown-unknown`.
- Markdown - operator docs and knowledge packages are first-class artifacts in `SKILL.md`, `install.md`, `docs/*.md`, `domains/<site>/skill.md`, and `interaction-skills/*.md`.
- Shell/YAML - repository maintenance and CI are defined in `scripts/scan_sensitive.sh` and `.github/workflows/ci.yml`.
- Python - not an active runtime layer; only optional subprocess wrapper guidance exists in `docs/python-integration.md`.
## Runtime
- Rust stable toolchain - CI installs stable via `.github/workflows/ci.yml`; no `rust-toolchain` file is detected in the repo.
- Native CLI runtime - `browser-harness` routes commands to `bhctl` or `bhrun` in `rust/bins/browser-harness-cli/src/main.rs`.
- Long-lived daemon runtime - `bhd` owns the browser websocket and session state through `rust/bins/bhd/src/main.rs` and `rust/crates/bh-daemon/src/lib.rs`.
- Unix socket IPC - daemon files are `/tmp/bu-<name>.sock`, `/tmp/bu-<name>.pid`, and `/tmp/bu-<name>.log` from `rust/crates/bh-discovery/src/lib.rs`; `bhrun` connects through `std::os::unix::net::UnixStream` in `rust/bins/bhrun/src/main.rs`.
- Browser runtime - local Chrome, Chromium, and Microsoft Edge profiles are discovered via `DevToolsActivePort` in `rust/crates/bh-discovery/src/lib.rs`; remote Browser Use sessions are supported through `rust/crates/bh-remote/src/lib.rs`.
- WASM guest runtime - `bhrun run-guest` and `bhrun serve-guest` execute `.wat`/`.wasm` guests through Wasmtime in `rust/bins/bhrun/src/main.rs` and `rust/crates/bh-wasm-host/src/lib.rs`.
- Cargo - workspace root is `rust/Cargo.toml`; all runtime crates and binaries are Cargo workspace members.
- Lockfile: present at `rust/Cargo.lock`; use it as the dependency source of truth for resolved crate versions.
## Frameworks
- Tokio `1.52.1` - async runtime and Unix listener support for daemon/control paths in `rust/Cargo.toml`, `rust/crates/bh-daemon/src/lib.rs`, and `rust/bins/bhctl/src/main.rs`.
- Wasmtime `30.0.2` - WASM guest execution engine used by `rust/bins/bhrun/src/main.rs` and declared in `rust/Cargo.toml`.
- tokio-tungstenite `0.24.0` - CDP websocket transport in `rust/crates/bh-cdp/src/lib.rs`.
- reqwest `0.12.28` with `json` and `rustls-tls` - Browser Use API client and runner `http-get` implementation in `rust/crates/bh-remote/src/lib.rs` and `rust/bins/bhrun/src/main.rs`.
- serde `1.0.228` and serde_json `1.0.149` - JSON protocol serialization across `rust/crates/bh-protocol/src/lib.rs`, `rust/crates/bh-wasm-host/src/lib.rs`, and all CLI payload handling.
- Rust built-in test harness - run all unit tests with `cargo test --workspace --manifest-path rust/Cargo.toml` as documented in `docs/development.md` and `.github/workflows/ci.yml`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yangchuansheng/browser-harness-rust](https://github.com/yangchuansheng/browser-harness-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
