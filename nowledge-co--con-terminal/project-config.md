---
trigger: always_on
description: con is an open-source, GPU-accelerated terminal emulator with a built-in AI agent harness. Built in Rust.
---

# con — Development Guide

## What is con?

con is an open-source, GPU-accelerated terminal emulator with a built-in AI agent harness. Built in Rust.

- **macOS** — primary target, shipped. Metal-backed libghostty, signed DMG, Sparkle auto-update.
- **Windows** — first beta shipped as `v0.1.0-beta.25`. D3D11/DirectWrite renderer over ConPTY + libghostty-vt, unsigned ZIP distribution, notify-only update checker. Plan + open work: `docs/impl/windows-port.md`; status tracker: issue #34.
- **Linux** — planned; tracker: issue #18.

## Stack

- **UI**: upstream Zed GPUI (git dependency on `zed-industries/zed`, Apache 2.0). Windows backend is D3D11/DirectComposition; HWND child-embedding is the known gap for the Windows port.
- **Terminal runtime**: libghostty — full Ghostty terminal via C API, Metal GPU rendering, embedded as native NSView. macOS-only today; see `docs/impl/windows-port.md` for the Windows strategy (likely `libghostty-vt` + ConPTY + custom renderer).
- **Terminal FFI**: con-ghostty crate — thin Rust wrapper over libghostty C API (surface lifecycle, action callbacks, clipboard, key/mouse input). Cfg-gated to macOS; on other targets the crate compiles to an empty shell so the workspace resolves.
- **Terminal support crate**: con-terminal — theme and palette helpers only
- **AI agent**: Rig v0.34.0 (from crates.io, 13 providers, Tool trait)
- **Socket API**: JSON-RPC 2.0 with a platform-specific transport — Unix domain sockets on Unix, Windows Named Pipes (`\\.\pipe\con`) on Windows. Served by the app and consumed first by `con-cli`.

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
# Prerequisites: rust (stable, edition 2024), cmake, zig (for libghostty on macOS)
cargo build            # debug (macOS)
cargo build --release  # release (macOS)
cargo run -p con       # run the terminal (macOS)
cargo test --workspace # test
```

The `con` UI binary builds on macOS, Linux, and Windows. On non-macOS
targets the terminal pane uses a placeholder view ("backend under
construction") until the Windows/Linux backend lands — the rest of the
app (agent panel, settings, command palette, control socket at
`\\.\pipe\con` on Windows / `/tmp/con.sock` on Unix) is fully wired.
See `docs/impl/windows-port.md` for the porting plan and the path to
a working terminal on Windows.

```bash
# Windows (from a Developer Command Prompt for VS 2022; needs Zig 0.15.2+ on PATH
# for libghostty-vt; the binary ships as `con-app.exe` because `CON` is a
# reserved DOS device name):
cargo wbuild -p con --release          # produces target\release\con-app.exe
cargo wrun   -p con
cargo wtest  -p con-core -p con-cli -p con-agent -p con-terminal

# Linux (needs the GPUI linux runtime deps — see .github/workflows/ci-portable.yml):
cargo build -p con --release
```

The `w*` aliases (declared in `.cargo/config.toml`) wrap the
`--no-default-features --features con/bin-con-app` incantation the
Windows-named binary requires.

## Control Plane

- `con-cli` is a real client for Con's local control socket, not a stub.
- Implementation details live in `docs/impl/socket-api.md`.
- The current live E2E workflow lives in `docs/impl/con-cli-e2e.md`.

## Local Skill

- Project-local skill: `skills/con-cli-e2e/SKILL.md`
- Use it when validating the control plane from an external agent or when writing eval automation against a real running Con session.
- The skill expects agents to prefer `con-cli --json`, verify pane capabilities before acting, and treat `panes create` as provisional until the new pane reports as alive and shell-ready.

## Design Language


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nowledge-co/con-terminal](https://github.com/nowledge-co/con-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
