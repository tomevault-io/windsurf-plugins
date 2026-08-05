---
trigger: always_on
description: vu is an open-source, GPU-accelerated terminal emulator with a built-in AI agent harness. Built in Rust.
---

# vu — Development Guide

## What is vu?

vu is an open-source, GPU-accelerated terminal emulator with a built-in AI agent harness. Built in Rust.

- **macOS** — primary target, shipped. Metal-backed libghostty, signed DMG, Sparkle auto-update.
- **Windows** — first beta shipped as `v0.1.0-beta.25`. D3D11/DirectWrite renderer over ConPTY + libghostty-vt, unsigned ZIP distribution, notify-only update checker. Plan + open work: `docs/impl/windows-port.md`; status tracker: issue #34.
- **Linux** — preview. Real PTY pane via `libghostty-vt`, styled-cell paint (SGR colors / bold / italic / underline / inverse + block cursor), client-side decorations with the same caption cluster Windows uses, transparent ARGB window with rounded corners, and KWin-Wayland backdrop blur via `org_kde_kwin_blur` where the compositor exposes it. Plan + open work: `docs/impl/linux-port.md`; tracker: issue #18.

## Stack

- **UI**: upstream Zed GPUI (git dependency on `zed-industries/zed`, Apache 2.0). Windows backend is D3D11/DirectComposition; HWND child-embedding is the known gap for the Windows port.
- **Terminal runtime**: libghostty — full Ghostty terminal via C API, Metal GPU rendering, embedded as native NSView. macOS uses the full embedded libghostty; Windows and Linux consume the carved-out `libghostty-vt` parser instead and pair it with their own renderers (D3D11/DirectWrite on Windows, GPUI per-row `StyledText` on the Linux preview today / GPUI-owned glyph-atlas grid renderer in the long term).
- **Terminal FFI**: vu-ghostty crate — thin Rust wrapper over libghostty C API on macOS (surface lifecycle, action callbacks, clipboard, key/mouse input). On Windows + Linux it wraps `libghostty-vt` plus per-platform PTY (`ConPTY` / Unix PTY) and renderer plumbing. Per-platform code lives in `vu-ghostty/src/{terminal,windows,linux}/`; the workspace consumes the same `GhosttyApp` / `GhosttyTerminal` / `TerminalColors` type names from each.
- **Terminal support crate**: vu-terminal — theme and palette helpers only
- **AI agent**: Rig v0.40.0 (from crates.io, multi-provider clients, Tool and AgentHook traits)
- **Socket API**: JSON-RPC 2.0 with a platform-specific transport — Unix domain sockets on Unix, Windows Named Pipes (`\\.\pipe\vu`) on Windows. Served by the app and consumed first by `vu-cli`.

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
│   ├── vu/           # Main binary (GPUI app shell)
│   ├── vu-core/      # Shared logic (harness, config, session)
│   ├── vu-terminal/  # Terminal themes and palette helpers
│   ├── vu-ghostty/   # Ghostty FFI wrapper — primary macOS backend (libghostty C API)
│   ├── vu-agent/     # AI harness (Rig 0.40, tools, conversation)
│   └── vu-cli/       # CLI + socket client for the live local control plane
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
# Prerequisites: rust (stable, edition 2024), cmake, Zig 0.15.2 exactly (for libghostty / libghostty-vt)
cargo build            # debug (macOS)
cargo build --release  # release (macOS)
cargo run -p vu       # run the terminal (macOS)
cargo test --workspace # test
```

The `vu` UI binary builds on macOS, Linux, and Windows. macOS uses
the embedded full libghostty + Metal renderer; Windows ships a
ConPTY + libghostty-vt + D3D11/DirectWrite renderer; Linux ships a
Unix PTY + libghostty-vt + GPUI-owned `StyledText` paint path. The
agent panel, settings, command palette, and control socket
(`\\.\pipe\vu` on Windows, `/tmp/vu.sock` on Unix) are fully
wired on every platform. See `docs/impl/windows-port.md` and
`docs/impl/linux-port.md` for the per-platform porting plans and
the path to the long-term GPU-accelerated grid renderer on each
non-macOS target.

```bash
# Windows (from a Developer Command Prompt for VS 2022; needs Zig 0.15.2 exactly on PATH
# for libghostty-vt; release scripts retain the `vu-app.exe` Windows alias):
cargo wbuild -p vu --release          # produces target\release\vu-app.exe
cargo wrun   -p vu
cargo wtest  -p vu-core -p vu-cli -p vu-agent -p vu-terminal

# Linux (needs the GPUI linux runtime deps — see .github/workflows/ci-portable.yml):
cargo build -p vu --release
```

The `w*` aliases (declared in `.cargo/config.toml`) wrap the
`--no-default-features --features vu/bin-vu-app` incantation the
Windows release alias uses.

## Control Plane

- `vu-cli` is a real client for Vu's local control socket, not a stub.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cdbkk/vu](https://github.com/cdbkk/vu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
