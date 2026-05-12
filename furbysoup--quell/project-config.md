---
trigger: always_on
description: A Windows-native terminal proxy (and eventually standalone terminal) for AI CLI tools that eliminates scroll-jumping and flicker. It intercepts child process VT output via ConPTY, tracks screen state with a VT100 emulator, and sends only differential updates to the display.
---

# Quell — Project Guide

## What This Is

A Windows-native terminal proxy (and eventually standalone terminal) for AI CLI tools that eliminates scroll-jumping and flicker. It intercepts child process VT output via ConPTY, tracks screen state with a VT100 emulator, and sends only differential updates to the display.

## Research

Prior research lives in `research/` (gitignored, local only) organized by topic. Check `research/INDEX.md` for the full index.

## Architecture

```
User's Terminal ←→ quell (proxy) ←→ ConPTY ←→ AI CLI tool (e.g. Claude Code)
```

**Workspace layout:**
- `src/` — Shared engine library (used by CLI, GUI, tests, benches)
  - `proxy/` — Main proxy loop, I/O threads, event handling
  - `conpty/` — Windows ConPTY session management
  - `vt/` — VT100 emulation, sync block detection, differential rendering
  - `history/` — Scrollback history buffer with safe-replay filtering
  - `config/` — Configuration file loading and CLI args
- `cli/` — Phase 1 CLI binary (depends on `quell` library)
- `gui/` — Phase 2 Tauri GUI (placeholder, depends on `quell` library)

## Build & Run

```bash
cargo build                    # Debug build (production features only)
cargo build --release          # Release build
cargo build --features recording  # Dev build with VT recording support
cargo run -p quell-cli -- claude -- --dangerously-skip-permissions  # Run with Claude Code as child process
cargo test                     # Run all tests (default features)
cargo test --features recording  # Run all tests including recording
cargo test --test unit         # Unit tests only
cargo test --test integration  # Integration tests only
cargo bench                    # Run benchmarks
```

## Testing Requirements

Every feature needs unit tests + integration tests where applicable. Live-proving is tracked by automated hooks (see below).

**Test organization:**
- `tests/unit/` — Pure logic tests (VT parsing, diffing, sync detection, history filtering)
- `tests/integration/` — ConPTY spawning, pipe I/O, proxy end-to-end
- `benches/` — Performance benchmarks (VT diffing throughput, sync detection speed)

## Logging Standards

All modules use `tracing` with structured fields (not string interpolation). Levels: `error!` (unrecoverable), `warn!` (recovered), `info!` (lifecycle), `debug!` (frame-level), `trace!` (byte-level). Output to `logs/quell.log` and via `RUST_LOG` env var.

## Feature Flags

Dev-only functionality is gated behind Cargo feature flags to keep the production binary lean.

| Feature | Purpose | What it gates |
|---------|---------|---------------|
| `recording` | VT output capture for replay testing | `--record` CLI flag, `recorder.rs` module, `Proxy.recorder` field, hot-path recording hook |

**Rules:**
- Default features are empty — `cargo build` produces a clean production binary
- Dev/test builds use `cargo build --features recording` or `cargo test --features recording`
- **Never add runtime dependencies (Cargo.toml `[dependencies]`) for feature-gated code.** Use only stdlib. If a dep is truly needed, make it optional: `foo = { version = "X", optional = true }` and add it to the feature's dep list.
- **Never add unconditional code to the hot path** (the `recv(output_rx)` loop in `proxy/mod.rs`) for dev-only features. All hot-path additions must be behind `#[cfg(feature = "...")]`.
- New dev-only features must follow this same pattern: feature flag in `Cargo.toml`, `#[cfg]` on all production-path code.
- The release CI workflow (`cargo build --release`) must NOT enable dev features.

**Testing both configurations:**
```bash
cargo test                            # Default (no dev features) — must pass
cargo test --features recording       # With recording — must also pass
cargo clippy --lib                    # Default — no warnings
cargo clippy --lib --features recording  # With recording — no warnings
```

## Code Conventions

- **Rust 2024 edition** with stable toolchain
- **Error handling:** Use `anyhow::Result` for application code, `thiserror` for library errors
- **No unwrap() in non-test code** — use `?`, `.context()`, or explicit error handling
- **Naming:** snake_case for functions/variables, PascalCase for types, SCREAMING_SNAKE for constants
- **Module structure:** Each module has `mod.rs` with public API, internal files for implementation
- **Comments:** Only where the logic isn't self-evident. No boilerplate doc comments on obvious functions.

## UI/UX Conventions (Phase 2+)

- **Use the `/frontend-design` skill** when building or modifying GUI components to avoid generic "AI slop" aesthetics
- **Vanilla CSS with custom properties** for theming — no CSS frameworks. Single `theme.css` as source of truth.
- **Coordinate xterm.js theme object with CSS custom properties** — canvas-rendered terminal colors are independent of CSS, so the chrome and terminal palettes must be kept in sync manually
- **Dark theme defaults:** no pure black — use dark greys (`#1e1e1e` or tinted darks like `#0d1117`), desaturate bright colors, maintain WCAG 4.5:1 contrast minimum for body text

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FurbySoup/quell](https://github.com/FurbySoup/quell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
