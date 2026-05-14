---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rust terminal emulator library with Python 3.12+ bindings (PyO3). Provides VT100/VT220/VT320/VT420/VT520 compatibility with PTY support, streaming server, screenshot rendering, and graphics protocols (Sixel, iTerm2, Kitty).

**Sister Projects** (keep config files, CLI options, and features in sync):
- `../par-term-emu-tui-rust` - Python TUI application ([GitHub](https://github.com/paulrobello/par-term-emu-tui-rust), [PyPI](https://pypi.org/project/par-term-emu-tui-rust/))
- `../par-term` - Rust terminal frontend ([GitHub](https://github.com/probello/par-term))

## Build Commands

```bash
make setup-venv          # Initial setup: creates .venv, installs deps
make dev                 # Build library (release mode via maturin) - USE THIS, not cargo build
make dev-streaming       # Build with streaming feature enabled
make checkall            # All quality checks (run before every commit)
```

**Never use `cargo build` directly** for PyO3 modules - it will fail at link stage. Always use `make dev` (maturin).

### Running Tests

```bash
make test                # Run all tests (Rust + Python)
make test-rust           # Rust tests only
make test-python         # Python tests only (rebuilds first)

# Single Rust test
cargo test --lib --no-default-features --features pyo3/auto-initialize test_name

# Single Python test file
uv run pytest tests/test_terminal.py -v

# Single Python test function
uv run pytest tests/test_terminal.py::test_function_name -v

# Rust streaming tests
cargo test --lib --no-default-features --features pyo3/auto-initialize,streaming test_name
```

The `--no-default-features --features pyo3/auto-initialize` flags are required because:
- Default feature enables `extension-module` which prevents linking during tests
- `auto-initialize` bootstraps the Python interpreter for Rust test environment

### Code Quality

```bash
make lint                # Rust clippy + fmt (auto-fix)
make lint-python         # Python ruff format + check + pyright
make fmt                 # Rust format only
make fmt-python          # Python format only
```

### Streaming Server & Web Frontend

```bash
make streamer-run        # Build + run streaming server on ws://127.0.0.1:8099
make streamer-run-http   # With HTTP server serving web_term/
make web-build-static    # Build Next.js frontend → web_term/ (run after frontend changes)

# Proto regeneration (after editing proto/terminal.proto)
make proto-rust          # Generate terminal.pb.rs
make proto-typescript    # Generate TypeScript proto code
make web-build-static    # Rebuild frontend with new proto
```

## Architecture

### Crate Structure

The crate produces three artifacts:
- **Python extension** (`cdylib`): Built via maturin, exposes `par_term_emu_core_rust._native` module
- **Rust library** (`rlib`): For use by other Rust projects (e.g., `par-term`)
- **Streaming server binary** (`par-term-streamer`): Requires `streaming` feature flag

### Feature Flags

| Feature | Purpose |
|---------|---------|
| `python` (default) | PyO3 bindings with `extension-module` |
| `streaming` | WebSocket server, protobuf, TLS, HTTP, CLI deps |
| `rust-only` | No Python bindings |
| `full` | `python` + `streaming` |
| `regenerate-proto` | Rebuild protobuf from `proto/terminal.proto` |
| `jemalloc` | Better server performance (non-Windows) |

### Data Flow

```
Input bytes → VTE Parser → Perform trait callbacks → Terminal state (Grid/Cursor) → Python API queries
                           (src/terminal/sequences/)   (src/terminal/mod.rs)        (src/python_bindings/)
```

### Key Source Layout

- `src/terminal/mod.rs` - Main `Terminal` struct with all state
- `src/terminal/sequences/{csi,osc,esc,dcs}.rs` - VTE escape sequence handlers
- `src/terminal/write.rs` - Character writing logic
- `src/terminal/trigger.rs` - Regex-based output pattern matching
- `src/grid.rs` - 2D terminal buffer with scrollback (flat Vec, row-major)
- `src/pty_session.rs` - PTY session with background reader thread
- `src/python_bindings/` - PyO3 wrappers (`terminal.rs`, `pty.rs`, `streaming.rs`, `types.rs`, `enums.rs`)
- `src/streaming/` - WebSocket streaming protocol
- `src/screenshot/` - Terminal-to-image rendering (embedded JetBrains Mono + Noto Emoji fonts)
- `src/graphics/` - Unified Sixel/iTerm2/Kitty graphics (all normalized to `TerminalGraphic` with RGBA)
- `src/lib.rs` - Module declarations, re-exports, and `_native` PyO3 module registration

### Streaming Protocol Layers

When modifying the streaming protocol, changes flow through 3 layers:

1. **`proto/terminal.proto`** → `src/streaming/terminal.pb.rs` (generated, don't edit directly)
2. **`src/streaming/protocol.rs`** - App-level types (`ServerMessage`, `ClientMessage`, `EventType` enums)
3. **`src/streaming/proto.rs`** - Conversion between app types ↔ protobuf wire format

Also update:
- `src/python_bindings/streaming.rs` - Dict conversion + event type matching
- `tests/test_streaming.rs` - Integration tests (use `..` in destructuring for forward compat)
- `src/streaming/server.rs` - `build_connect_message()` helper when extending `Connected`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulrobello/par-term-emu-core-rust](https://github.com/paulrobello/par-term-emu-core-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
