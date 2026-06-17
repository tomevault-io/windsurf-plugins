---
trigger: always_on
description: This document provides context, patterns, and guidelines for AI coding assistants working in this repository. For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md).
---

# AGENTS.md

This document provides context, patterns, and guidelines for AI coding assistants working in this repository. For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md).

## Product Overview

**Strands Shell** is a Bourne-compatible shell for AI agents that runs entirely in-process. It implements a complete operating-system environment inside a single userspace process — inspired by BusyBox and Toybox — but it never calls `fork`/`exec` or makes direct system calls. Every operation flows through a pluggable `Kernel` trait, giving callers fine-grained control over what an agent can see and do (files, network domains, credentials) without containers, microVMs, or firewalls.

It is a Rust crate that compiles to several targets from one source of truth:

- **Native binary** (`strands-shell`) and a Rust library
- **Python** extension module (`strands-shell` on PyPI), via [PyO3](https://pyo3.rs/) + [maturin](https://www.maturin.rs/)
- **Node.js** native addon (`@strands-agents/shell` on npm), via [napi-rs](https://napi.rs/)
- **WASM** module targeting `wasm32-wasip2`

## Architecture

The crate is the single source of truth; every binding wraps the same core.

- **`Kernel` trait (`src/os.rs`) is the security boundary.** All filesystem, process, and network effects go through a `Kernel` implementation. There is no `fork`/`exec` and no direct syscalls. The bundled implementation is `VfsKernel` (`src/vfs_kernel.rs`), backed by an in-process virtual filesystem (`src/vfs.rs`); callers can supply their own (S3-backed, database-backed, etc.) via `Shell::with_kernel()`.
- **In-process VFS with binds.** Directories are mounted into the VFS as *binds* — `copy` mode snapshots files into the VFS at build time; `direct` mode passes reads/writes through to the host, mediated by the kernel. Bind configuration lives in `src/vfs_config.rs`.
- **Commands are split in two:**
  - **Builtins** in `src/builtins/` — things that mutate shell state (`cd`, `export`, `alias`, `set`, control-flow helpers, etc.). They are dispatched by name in `src/builtins/mod.rs` (`lookup()` matches the builtin name to its function).
  - **Isolated commands** in `src/commands/` — coreutils-style programs (`cat`, `grep`, `sed`, `curl`, `jq`, …) that take a kernel + args and produce output. Each is registered with the `#[command("name")]` proc-macro from `strands-shell-macros`.
- **Parser / executor:** `src/parser.rs` parses shell syntax; `src/exec.rs` evaluates it (pipelines, redirections, expansions, control flow).
- **Bindings:** `src/python.rs` (PyO3, the `strands_shell._native` module) and `src/js.rs` (napi-rs). They are intentionally parallel in shape — keep them in sync semantically. The customer-facing Python surface (the `Shell`, `Bind`, `Cred`, `Limits` classes and typed errors) lives in the pure-Python wrapper at `python/strands_shell/__init__.py`.
- **WASM entry:** `src/wasm_main.rs` reads commands from WASI stdin and writes to WASI stdout/stderr; each instance runs in isolated linear memory.
- **MCP:** `src/mcp.rs` is the built-in MCP *server* (exposes the shell as tools); `src/mcp_client.rs` is the MCP *client* (servers configured under `[[mcp]]` become Lua modules).

### The `#[command(...)]` macro

A new isolated command is a function annotated with the proc-macro:

```rust
#[command("ls")]
async fn cmd_ls(os: &dyn Kernel, args: &[String]) -> i32 {
    // ...
}
```

The macro (defined in `strands-shell-macros/src/lib.rs`) registers the command via `inventory` on native targets and feeds the static lookup table used on WASM. Builtins are *not* registered this way — add them to the `match` in `src/builtins/mod.rs`.

## Build & Test Commands

Use the same commands as [CONTRIBUTING.md](./CONTRIBUTING.md#development-environment) so they don't drift. The Rust toolchain is required for every workflow because all bindings build from the crate.

### Rust (shell core)

```bash
cargo build                              # build the library and binaries
cargo test --workspace --all-targets     # unit + integration tests
cargo fmt                                # format
cargo clippy --workspace --all-targets   # lint
cargo doc --workspace --no-deps --open   # API reference
```

Integration tests live in `tests/`: `shell_integration.rs`, `curl_integration.rs`, `lua_integration.rs`, `mcp_integration.rs`, `vfs_unit.rs`.

### Python bindings

```bash
python -m venv .venv && source .venv/bin/activate
pip install maturin pytest
maturin develop --features python        # build + install into the venv
pytest tests/python -v                   # run the Python test suite
```

Python sources are under `python/strands_shell/`; the compiled module is `strands_shell._native`. Tests: `tests/python/*.py`.

### Node.js bindings

```bash
npm install            # install dependencies
npm run build          # release build of the native addon
npm run build:debug    # faster debug build for local development
npm test               # run the Node.js test suite (tests/js/*.mjs)
```

### WASM module

```bash
./scripts/build-wasm.sh --release        # needs wasi-sdk >= 32
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strands-agents/shell](https://github.com/strands-agents/shell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
