---
trigger: always_on
description: con is an open-source, macOS-native, GPU-accelerated terminal emulator with a built-in AI agent harness. Built in Rust.
---

# con — Development Guide

## What is con?

con is an open-source, macOS-native, GPU-accelerated terminal emulator with a built-in AI agent harness. Built in Rust.

## Stack

- **UI**: GPUI-CE v0.3.3 (community edition of Zed's framework, Apache 2.0)
- **Terminal runtime**: libghostty — full Ghostty terminal via C API, Metal GPU rendering, embedded as native NSView
- **Terminal FFI**: con-ghostty crate — thin Rust wrapper over libghostty C API (surface lifecycle, action callbacks, clipboard, key/mouse input)
- **Terminal support crate**: con-terminal — theme and palette helpers only
- **AI agent**: Rig v0.34.0 (from crates.io, 13 providers, Tool trait)
- **Socket API**: Unix domain sockets + newline-delimited JSON-RPC, served by the app and consumed first by `con-cli`

## Repository Layout

```
kingston/
├── DESIGN.md          # Architecture and design decisions
├── CLAUDE.md          # This file — development guide
├── docs/
│   ├── impl/          # Implementation notes per crate/subsystem
│   └── study/         # Research notes on 3pp dependencies
├── postmortem/        # Issue postmortems (YYYY-MM-DD-title.md)
├── crates/
│   ├── con/           # Main binary (GPUI app shell)
│   ├── con-core/      # Shared logic (harness, config, session)
│   ├── con-terminal/  # Terminal themes and palette helpers
│   ├── con-ghostty/   # Ghostty FFI wrapper — primary macOS backend (libghostty C API)
│   ├── con-agent/     # AI harness (Rig 0.34, tools, conversation)
│   └── con-cli/       # CLI + socket client for the live local control plane
├── postmortem/        # Integration & incident postmortems
├── assets/            # Themes, fonts, icons
└── 3pp/               # Third-party source (READ-ONLY reference, .gitignored)
```

## 3pp Policy

The `3pp/` directory contains third-party source checkouts for **read-only reference only**. It is `.gitignored` — never modify, commit, or depend on files in `3pp/`.

- All third-party dependencies come from **crates.io** (or git URLs in Cargo.toml).
- If a 3pp library has a bug, **upstream the fix** to the library's GitHub repo. Do not patch locally.
- `3pp/` exists solely so you can read and study how dependencies work internally.

## Build

```bash
# Prerequisites: rust (stable, edition 2024), cmake
cargo build            # debug
cargo build --release  # release
cargo run -p con       # run the terminal
cargo test --workspace # test

# GPUI needs runtime_shaders feature (already set)
# con currently ships the embedded Ghostty runtime on macOS
```

## Control Plane

- `con-cli` is a real client for Con's local control socket, not a stub.
- Implementation details live in `docs/impl/socket-api.md`.
- The current live E2E workflow lives in `docs/impl/con-cli-e2e.md`.

## Local Skill

- Project-local skill: `skills/con-cli-e2e/SKILL.md`
- Use it when validating the control plane from an external agent or when writing eval automation against a real running Con session.
- The skill expects agents to prefer `con-cli --json`, verify pane capabilities before acting, and treat `panes create` as provisional until the new pane reports as alive and shell-ready.

## Design Language

- **Font**: IoskeleyMono (embedded, all weights) for terminal chrome — tabs, sidebar, input bar. System font (.SystemUIFont / SF Pro) for AI panel prose text, settings panel, and any non-terminal UI. Code blocks and terminal previews use IoskeleyMono via `mono_font.family` in theme JSON.
- **Default theme**: Flexoki Light. Dark available as Flexoki Dark.
- **Icons**: Phosphor Icons only (phosphoricons.com). Copy SVGs from `3pp/phosphor-icons/SVGs/regular/` into `assets/icons/phosphor/`. Never draw icons manually — always use the Phosphor library. Reference as `"phosphor/icon-name.svg"` in code. **Critical**: always set `.text_color()` directly on every `svg()` element — parent container color does NOT propagate to SVG stroke colors in GPUI.
- **Borderless**: No `border_1()`, `border_r_1()`, etc. Use opacity-based fills for surface separation.
- **Shadowless**: No `shadow_sm()`, `shadow_lg()`, etc. Use bg opacity for elevation.
- **Color by meaning only**: Monochrome surfaces by default. Accent color for semantic states (focus, active, warning, error).
- **Typography as hierarchy**: Size, weight, and opacity create structure — not boxes and borders.

See `docs/design/con-design-language.md` for full design system.

## UI/UX Principles

These principles govern all UI iteration. Follow them proactively — don't wait for the user to point out violations.

### Use gpui-component library first

Before building custom UI, check `3pp/gpui-component/` for an existing component. The library has 60+ components including:

- **Select** (`select::Select`, `SelectState<SearchableVec<String>>`) — searchable dropdowns. Use for any list selection instead of hand-rolling clickable divs.
- **Button** (`button::Button`) — use `.ghost()`, `.primary()`, `.small()`, `.icon()` variants. Never hand-roll clickable divs for buttons.
- **Input** (`input::Input`, `InputState`) — text fields with `.appearance(false)` for inline, `.cleanable()`, `.placeholder()`.
- **Switch** (`switch::Switch`) — toggles. Never hand-roll toggle divs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nowledge-co/con](https://github.com/nowledge-co/con) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
