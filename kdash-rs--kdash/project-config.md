---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) and other AI Agents when working with code in this repository.
---

# CLAUDE.md/AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) and other AI Agents when working with code in this repository.

## Project Overview

KDash is a terminal UI dashboard for Kubernetes, built with Rust using ratatui (TUI framework) and the kube-rs client library. It provides real-time monitoring of Kubernetes resources with keyboard-driven navigation.

## Build & Development Commands

```bash
# Build (runs lint + tests first)
make build

# Run (formats, lints, then runs)
make run

# Run directly without checks
cargo run

# Run with CLI args (e.g., tick rate, debug mode)
cargo run -- -t 100 -d=debug

# Lint (clippy with strict warnings-as-errors)
cargo clippy --all --all-features --all-targets --workspace -- -D warnings
# or
make lint

# Format
cargo fmt

# Run all tests (lint + cargo test)
make test

# Run a single test
cargo test <test_name>

# Run tests in a specific module
cargo test <module>::tests

# Test coverage (requires cargo-tarpaulin)
make test-cov
```

## Manual / UAT testing (TUI harness)

`scripts/uat/harness.py` drives the live TUI for end-to-end / feature testing.
It launches `kdash` in a PTY, renders the real ratatui screen with `pyte`, sends
scripted keystrokes, and asserts on the on-screen text. Use it to verify a
feature actually works in the running app, not just in unit tests.

```bash
# one-time: venv with the two deps
python3 -m venv /tmp/kdash-uat-venv && /tmp/kdash-uat-venv/bin/pip install pyte pexpect

cargo build
/tmp/kdash-uat-venv/bin/python scripts/uat/harness.py <program> <outdir> [binary] [args...]
# e.g.
/tmp/kdash-uat-venv/bin/python scripts/uat/harness.py scripts/uat/example.prog /tmp/out
```

- Program = a step file (`key:`, `type:`, `expect:`, `refute:`, `snap:`, `wait:`,
  `settle`, `spawn:`); full step + key-name reference in `scripts/uat/README.md`.
- Exit code is non-zero if any `expect`/`refute` fails, so it slots into CI.
- The harness replies to crossterm's cursor-position query (`ESC[6n`); without
  that reply the app aborts with "cursor position could not be read".
- **Write actions** (delete/scale/cordon/suspend/trigger) are best verified by
  their cluster side-effect via `kubectl`, not just the rendered screen.
- The scale input modal **prefills the current replica count** — clear it with
  `backspace` before typing a new value.
- `docs/` is gitignored; keep UAT plans/reports/screens there as local scratch.

## Architecture

The app follows an async event-driven architecture with three main communication channels (tokio mpsc):

### Core Loop (main.rs)
- **UI thread** (main runtime): terminal rendering loop using crossterm + ratatui. Polls for input/tick/kubeconfig-change events.
- **Network thread** (separate OS thread + tokio runtime): runs three concurrent tasks:
  - `Network` — handles one-shot K8s API calls (list pods, get nodes, etc.) via `IoEvent`
  - `NetworkStream` — handles streaming operations (log tailing, exec) via `IoStreamEvent`
  - `CmdRunner` — runs kubectl shell commands (describe, top) via `IoCmdEvent`

### Module Responsibilities

- **`app/`** — Application state (`App` struct) and Kubernetes resource models. Each resource type (pods, deployments, nodes, etc.) has its own file defining its `Kube*` data struct and an `*Resource` trait impl. `models.rs` contains shared UI state types (`StatefulTable`, `ScrollableTxt`, `TabsState`, etc.). `key_binding.rs` defines all keybinding actions.
- **`network/`** — K8s API interaction layer. `mod.rs` has `Network` struct handling `IoEvent` variants. `stream.rs` handles streaming events. Uses `kube` crate client.
- **`ui/`** — Rendering logic. `draw()` in `mod.rs` is the entry point. `overview.rs` renders the main dashboard. `resource_tabs.rs` renders resource-specific views. `theme.rs` handles color theming.
- **`handlers/`** — Input event handling. Maps key presses and mouse events to app state changes and network dispatches. Uses `handle_workload_action!` macro for common resource interactions.
- **`event/`** — Terminal event abstraction (keyboard, mouse, tick, kubeconfig file watch).
- **`cmd/`** — Shell command execution (kubectl describe, top, etc.).
- **`config.rs`** — User config from `$KDASH_CONFIG` or `~/.config/kdash/config.yaml` (YAML, supports keybinding and theme overrides).

### Key Patterns

- App state is `Arc<Mutex<App>>` shared between UI and network threads.
- Network calls are dispatched by sending `IoEvent`/`IoStreamEvent`/`IoCmdEvent` through channels; results are written back to `App` state under the mutex.
- Navigation uses a stack (`nav_stack`) of `Route` objects with `ActiveBlock` enum variants.
- Each K8s resource type follows a consistent pattern: data struct in `app/<resource>.rs`, network fetch in `network/mod.rs`, UI rendering in `ui/resource_tabs.rs`, key handling in `handlers/mod.rs`.

### Resource Actions (write path)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kdash-rs/kdash](https://github.com/kdash-rs/kdash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
