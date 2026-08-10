---
trigger: always_on
description: This file explains how humans and coding agents should work in **GPUI Starter**. Start with the beginner workflow if the project is unfamiliar; use the advanced guidance when changing architecture, state, reusable components, or window behavior.
---

# Repository Guidelines

This file explains how humans and coding agents should work in **GPUI Starter**. Start with the beginner workflow if the project is unfamiliar; use the advanced guidance when changing architecture, state, reusable components, or window behavior.

## Project at a Glance

GPUI Starter is a small Rust 2021 workspace for a native desktop application built with [GPUI](https://www.gpui.rs/).

```text
.
├── Cargo.toml                 # Workspace members and shared dependencies
├── justfile                   # Short development commands
├── crates/
│   ├── desktop/               # Native application bootstrap
│   │   ├── assets/            # Application icon files
│   │   └── src/main.rs        # Application, menu, keybindings, and window setup
│   └── ui/                    # Reusable UI and application state
│       ├── src/lib.rs         # Public API and RootView
│       ├── src/button.rs      # Button component and variants
│       ├── src/theme.rs       # Runtime theme tokens
│       └── themes/groknight.json # Bundled/reference Groknight palette
└── README.md                  # Short project overview
```

The workspace contains two crates:

| Crate | Package | Responsibility |
| --- | --- | --- |
| `crates/desktop` | `gpui-starter-desktop` | Starts GPUI, registers global actions, configures the native window, and mounts the root view. |
| `crates/ui` | `gpui-starter-ui` | Owns views, components, local UI state, styling, and theme tokens. |

**Ownership rule:** put behavior in the crate that owns it. Window and application lifecycle code belongs in `desktop`; reusable visual behavior belongs in `ui`. Expose cross-crate APIs through the owning crate's `src/lib.rs` instead of reaching into private modules.

## Beginner Quick Start

### Prerequisites

Install:

- A current stable Rust toolchain through [rustup](https://rustup.rs/).
- [`just`](https://github.com/casey/just) for the repository shortcuts.
- The native build tools required by Rust and GPUI for your operating system.

Confirm the tools are available:

```sh
rustc --version
cargo --version
just --version
```

### Run the application

```sh
just dev
```

The app opens a `960 × 640` window containing the button-variant demo. Quit with **⌘Q** on macOS or **Ctrl+Q** on other supported platforms.

### Make a first change

A safe first change is editing the empty-state label in `crates/ui/src/lib.rs`:

```rust
let status: SharedString = if self.clicks == 0 {
    "Choose a button".into()
} else {
    format!("Clicked {n}×", n = self.clicks).into()
};
```

Then format, check, and run the app:

```sh
cargo fmt --all
just check
just dev
```

For a visible change, manually verify normal, hover, active, and click behavior before considering the work complete.

## Development Commands

| Command | Purpose | When to use it |
| --- | --- | --- |
| `just dev` | Run `gpui-starter-desktop` in debug mode. | Normal development and visual checks. |
| `just run` | Run the optimized release build. | Confirm release-mode behavior or performance. |
| `just build` | Build the desktop binary in release mode. | Produce or validate an optimized binary. |
| `just check` | Type-check both workspace crates. | Fast validation after Rust changes. |
| `cargo fmt --all` | Format all Rust code. | Before finishing any Rust edit. |
| `cargo fmt --all -- --check` | Verify formatting without changing files. | CI-style validation. |
| `cargo clippy --workspace --all-targets` | Run Rust lints across the workspace. | Before a PR or after non-trivial changes. |
| `cargo test --workspace` | Run workspace tests and test builds. | When behavior changes or tests are added. |

Use package-specific Cargo commands when iterating on one crate:

```sh
cargo check -p gpui-starter-ui
cargo check -p gpui-starter-desktop
```

Do not claim a command passed unless you actually ran it. If a command cannot run because of the local environment, report the command and the exact blocker.

## Recommended Workflow

1. **Read before editing.** Inspect the target module, its crate's `Cargo.toml`, and nearby patterns.
2. **Choose the owning crate.** Keep native lifecycle concerns in `desktop` and reusable UI concerns in `ui`.
3. **Make the smallest coherent change.** Avoid unrelated refactors, speculative abstractions, or new dependencies.
4. **Format and type-check.** Run `cargo fmt --all` and at least `just check`.
5. **Run focused validation.** Add tests for meaningful logic; run `cargo clippy --workspace --all-targets` for non-trivial work.
6. **Inspect the UI.** Run `just dev` for changes to layout, colors, interaction, window options, or rendering.
7. **Summarize precisely.** State what changed, why, what was validated, and any remaining limitations.

## Coding Style

Follow standard Rust and `rustfmt` conventions:

- Four-space indentation; never hand-align code against `rustfmt`.
- `snake_case` for modules, functions, variables, and tests.
- `PascalCase` for structs, enums, and traits.
- `SCREAMING_SNAKE_CASE` for constants.
- Prefer explicit, domain-specific names over abbreviations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lassejlv/gpui-starter](https://github.com/lassejlv/gpui-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
