---
trigger: always_on
description: The Rust toolchain version is pinned in [`src/rust-toolchain.toml`](../src/rust-toolchain.toml) to match what CI uses (currently 1.93). The pin is honored automatically by `rustup` — running any `cargo` command from `src/` (or below) downloads and selects that channel on first use. To opt out for one-off testing on a different toolchain, use `cargo +<channel> ...` or set `RUSTUP_TOOLCHAIN`. When bumping the pinned version, bump the matching `version: 'ms-prod-1.<N>'` lines in the two `.azure-pip
---

# MXC (Microsoft eXecution Container) — Copilot Instructions

## Prerequisites

The Rust toolchain version is pinned in [`src/rust-toolchain.toml`](../src/rust-toolchain.toml) to match what CI uses (currently 1.93). The pin is honored automatically by `rustup` — running any `cargo` command from `src/` (or below) downloads and selects that channel on first use. To opt out for one-off testing on a different toolchain, use `cargo +<channel> ...` or set `RUSTUP_TOOLCHAIN`. When bumping the pinned version, bump the matching `version: 'ms-prod-1.<N>'` lines in the two `.azure-pipelines/templates/*.Build.Job.yml` files in the same commit.

LSP servers are configured in `.github/lsp.json` for Rust and TypeScript. Install them before use:

```
rustup component add rust-analyzer
npm install -g typescript-language-server typescript
```

## Build Commands

### Full build (Windows)

```
build.bat                  # Release build for current architecture
build.bat --debug          # Debug build
build.bat --all            # Release build for both x64 and ARM64
build.bat --with-microvm   # Include NanVix micro-VM binaries
```

### Full build (Linux)

```
./build.sh                 # Release build
./build.sh --debug         # Debug build
./build.sh --rust-only     # Only Rust binaries, skip SDK/CLI
```

### Full build (macOS)

```
./build-mac.sh             # Release build for native architecture (seatbelt backend)
./build-mac.sh --debug     # Debug build
./build-mac.sh --all       # Build for both aarch64 and x86_64
./build-mac.sh --rust-only # Only Rust binaries, skip SDK
```

Requires Xcode Command Line Tools and Rust. Produces an unsigned `mxc-exec-mac` binary (codesigning + notarization happen at release time). Schema `0.7.0-alpha` or later required for macOS/Seatbelt backend.

### Individual components

```
# Rust workspace (from src/)
cargo build --release --target x86_64-pc-windows-msvc
cargo build --release --target aarch64-pc-windows-msvc
cargo build --release -p lxc          # Linux only — builds lxc-exec
cargo build --release -p mxc_darwin --target aarch64-apple-darwin  # macOS only — builds mxc-exec-mac

# SDK (from sdk/)
npm install && npm run build

# CLI (from cli/)
npm install && npm run build
```

### Lint and format

```
# Rust (from src/)
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings

# CLI (from cli/)
npx eslint src --ext .ts
```

### Tests

```
# Rust unit tests (from src/)
cargo test --workspace
cargo test -p wxc_common                    # Single crate
cargo test -p wxc_common -- config_parser   # Filter by test name

# SDK (from sdk/)
npm test
npm run test:integration

# CLI (from cli/) — requires build first
node --test dist/cli.test.js

# Local PowerShell helpers — run from repo root, require built binaries
tests\scripts\run_test_configs.ps1            # All test configs via wxc_test_driver
tests\scripts\run_basicprocess_test.ps1            # Single process container test
tests\scripts\run_isolation_session_tests.ps1                # IsolationSession one-shot E2E (requires host with the OS-side IsoSessionOps service)
tests\scripts\run_isolation_session_state_aware_tests.ps1    # IsolationSession state-aware lifecycle E2E (multi-invocation provision/start/exec/stop/deprovision, same host requirements)
tests\scripts\run_lxc_all_tests.sh            # All LXC tests (Linux)
tests\scripts\run_bwrap_all_tests.sh          # All Bubblewrap tests (Linux, requires bwrap)

# E2E test crate — Rust executor integration tests (from src/)
cargo test -p wxc_e2e_tests                 # Invokes MXC binaries directly
cargo test -p wxc_e2e_tests -- --ignored    # Include stress tests (run_on_repeat)
```

## Architecture

MXC is a **sandboxed code execution system** with a Rust core and TypeScript SDK/CLI layer.

### Containment backends

The Rust workspace (`src/`) implements multiple sandboxing backends behind the `ScriptRunner` trait (`core/wxc_common/src/script_runner.rs`):

| Backend | Binary | Platform | Module |
|---------|--------|----------|--------|
| AppContainer | `wxc-exec.exe` | Windows | `backends/appcontainer/common/src/appcontainer_runner.rs` |
| BaseContainer (OS sandbox API) | `wxc-exec.exe` | Windows | `backends/appcontainer/common/src/base_container_runner.rs` — calls `Experimental_CreateProcessInSandbox` via FlatBuffer |
| Windows Sandbox | `wxc-exec.exe` | Windows | `backends/windows_sandbox/common/src/windows_sandbox_runner.rs` |
| MicroVM (NanVix) | `wxc-exec.exe` | Windows | `backends/nanvix/runner/src/lib.rs` — feature-gated behind `microvm` |
| Hyperlight | `wxc-exec.exe` | Windows | `backends/hyperlight/common/src/lib.rs` — Hyperlight + Unikraft micro-VM backend |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/mxc](https://github.com/microsoft/mxc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
