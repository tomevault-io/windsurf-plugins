---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

muxterm is an iTerm-style terminal emulator for macOS (Rust, egui/eframe) where **every pane is backed by its own tmux session** on a dedicated socket (`tmux -L muxterm`). Quitting the app only detaches clients — sessions, processes, and scrollback survive; relaunch reattaches the saved layout. Requires `tmux` on the machine (`make setup` installs it via brew).

## Commands

```sh
cargo run --release          # run the app (also: make run)
cargo build                  # compile check
cargo test                   # all tests (root crate; unit tests live in #[cfg(test)] modules)
cargo test layout            # tests in one module
cargo test split_and_leaves  # single test by name
cargo run --bin mux -- peers # the mux CLI (second binary; default-run is the GUI)
make install                 # bundle muxterm.app (make app) + ship to /Applications + refresh ~/.cargo/bin/mux
```

The bundle recipe lives in the Makefile and `packaging/` (Info.plist template, icon generator); `assets/muxterm.icns` is checked in — regenerate with `make icon` only when the icon design changes.

`make cert` (once per machine, `packaging/make-signing-cert.sh`) creates a persistent self-signed `muxterm-local` code-signing identity in the login keychain **and trusts it for code signing** (`security add-trusted-cert`); `make app` signs with it when present (`SIGN_ID`), else ad-hoc. This is what stops macOS re-prompting for TCC "access data from other apps" on every rebuild (the agent-hook installer touches `~/.claude`/`~/.codex`/`~/.pi`/`~/.config/opencode` — other apps' dirs — and TCC pins the grant to the *designated requirement*, which for an ad-hoc signature is the ever-changing binary hash but for the cert is the stable bundle-id + cert leaf). The **trust** half is not optional: an untrusted self-signed cert is rejected by `spctl` and TCC won't anchor the grant on it, so it degrades back to the per-build hash and re-prompts — the exact failure the persistent cert is meant to prevent. The script is self-repairing: run `make cert` again on a machine whose `muxterm-local` cert exists but isn't trusted and it just adds the trust setting (then `tccutil reset SystemPolicyAppData dev.herval.muxterm` and re-Allow once).

There is no test binary/harness beyond inline unit tests, and no rustfmt/clippy config — match the existing hand-formatted style (notably `},` closing match arms).

## Architecture

Cargo workspace: the root `muxterm` crate plus `crates/egui_term`, a **vendored** terminal widget (egui + alacritty_terminal) carrying local patches (input gating, SGR mouse-wheel under tmux mouse mode, bracketed paste, IME, OSC 52 copy). Any change under `crates/egui_term/` must be recorded as a patch entry in `crates/egui_term/VENDOR.md`.

Two binaries share one library. `src/lib.rs` exposes only `agent`, `ask`, `layout`, `mesh`, and `state` — the modules used by both the GUI (`src/main.rs` + private modules) and the agent-mesh CLI (`src/bin/mux.rs`). Code needed by `mux` must live in one of those modules.

### The tmux trick (persistence)

- `tmux.rs` — each pane's PTY runs a tmux *client*: `tmux -L muxterm new-session -A -D -s mux-<8hex>`. `-A` makes fresh-spawn and restore-after-relaunch the same code path. Killing a session is an explicit step (cmd+w / shell exit); dropping a `Pane` merely detaches. The tmux.conf is regenerated at every launch.
- `state.rs` — layout (windows → tabs → split tree → session names) is saved to `state.json` on every mutation. On startup, `mux-*` sessions not referenced by the state are GC'd — **never when the state file failed to parse** (a corrupt state must not cost live sessions).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [herval/muxterm](https://github.com/herval/muxterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
