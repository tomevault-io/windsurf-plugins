---
trigger: always_on
description: > **Source of truth** for all contributors and AI agents working on this project.
---

# Horizon — Agent Guidelines

> **Source of truth** for all contributors and AI agents working on this project.

## Quick Start

An AI agent given these instructions should be able to go from zero to a running Horizon binary.

**Repository:** `https://github.com/peters/horizon.git`

### Option A — Download a release binary (fastest)

Pre-built binaries are attached to GitHub releases. Prefer the latest **non-prerelease** tag. Use `gh release download --repo peters/horizon` or download from `https://github.com/peters/horizon/releases/latest`.

| Platform | Asset | Contents |
|----------|-------|----------|
| Linux x64 | `horizon-linux-x64.tar.gz` | Single `horizon` binary — extract and make executable |
| macOS x64 | `horizon-osx-x64.tar.gz` | Single `horizon` binary — extract and make executable |
| Windows x64 | `horizon-windows-x64.exe` | Ready-to-run executable |

No Rust toolchain or system headers are needed for this path.

### Option B — Build from source

#### Prerequisites

- **Rust stable ≥ 1.88** (edition 2024). Install via [rustup](https://rustup.rs) if not present.
- **Linux only:** the eframe/wgpu rendering stack needs system headers. Install them before `cargo build`:
  - Debian/Ubuntu: `sudo apt install -y build-essential pkg-config libxkbcommon-dev libwayland-dev libxcb-render0-dev libxcb-shape0-dev libxcb-xfixes0-dev libvulkan-dev libgl-dev cmake`
  - Fedora: `sudo dnf install -y gcc pkg-config wayland-devel libxkbcommon-devel vulkan-loader-devel mesa-libGL-devel cmake`
  - Arch: `sudo pacman -S --needed base-devel wayland libxkbcommon vulkan-icd-loader cmake`
- **macOS:** Xcode Command Line Tools (`xcode-select --install`). Metal ships with the OS.
- **Windows:** MSVC build tools (installed automatically by `rustup` on the `msvc` target). DX12/Vulkan drivers ship with the GPU driver.

#### Build & Run

```bash
git clone https://github.com/peters/horizon.git
cd horizon
cargo run --release
```

#### Verify

```bash
cargo fmt --all -- --check
cargo test --workspace
cargo clippy --all-targets --all-features -- -D warnings
```

If any step fails, read the error, fix the prerequisite, and retry. On Linux, missing system headers are the most common issue — look for `pkg-config` or linker errors and install the corresponding `-dev` package.

## Project Overview

**Horizon** is a GPU-accelerated terminal board — a visual workspace for managing
multiple terminal sessions as freely positioned, resizable panels on a canvas.

**Stack:** Rust (edition 2024) · eframe/egui (wgpu backend) · alacritty_terminal (VT parsing, PTY, event loop)

## Workspace Layout

```
crates/
  horizon-core/       Core: terminal emulation, PTY, board & panel management
  horizon-ui/         Binary: eframe application, UI rendering, input handling
```

### horizon-core

- `error.rs` — Typed error enum via thiserror
- `terminal.rs` + `terminal/` — alacritty_terminal wrapper split by lifecycle, event handling, resize policy, selection, replay, and support helpers
- `panel.rs` — Panel = terminal + PTY session + identity
- `board.rs` + `board/tests/` — Board orchestration and colocated behavior-focused tests

### horizon-ui

- `main.rs` — Entry point, tracing init, eframe launch
- `app/` — `eframe::App` orchestration split by canvas, panels, sidebar, settings, session, persistence, workspace helpers, and focused action submodules
- `remote_hosts_overlay.rs` + `remote_hosts_overlay/` — remote SSH chooser state/input shell with dedicated query, layout, and paint helpers
- `terminal_widget/` — Terminal widget split by layout, input, render, scrollbar logic
- `input/` — Keyboard translation, mouse reporting, escape-sequence building
- `theme.rs` — Color palette (Catppuccin Mocha), styling constants

## Development Workflow

### Pre-push validation (all must pass)

```bash
cargo fmt --all -- --check
./scripts/check-maintainability.sh
RUSTFLAGS="-D warnings" cargo test --workspace
cargo clippy --all-targets --all-features -- -D warnings
cargo clippy --workspace --lib --bins --examples -- -D warnings -D clippy::unwrap_used -D clippy::expect_used
cargo clippy --workspace --all-targets --all-features -- -D warnings -W clippy::pedantic
```

- Run the validation commands in the exact checkout you will push. If you split work across branches or `git worktree`s, rerun the blocking and strict clippy tiers in each final branch/worktree after applying the split, not only in the original combined checkout.

### Configuration Changes

- When changing default presets, CLI flags, or any config-related code in `horizon-core/src/config.rs`, always sync the user's local config file (`~/.horizon/config.yaml`) to match

### Code Quality Bar

- Self-documenting code preferred over comments; add comments only for invariants, non-obvious tradeoffs, or safety contracts
- Use idiomatic Rust naming: `snake_case` (functions/modules), `CamelCase` (types), `SCREAMING_SNAKE_CASE` (consts)
- Typed error enums (thiserror) — no `Box<dyn Error>` or `.unwrap()` in library code
- Keep APIs explicit: prefer `Result<T, E>` and typed structs/enums over ad-hoc tuples

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peters/horizon](https://github.com/peters/horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
