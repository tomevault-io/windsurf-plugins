---
trigger: always_on
description: <!-- Keep in sync with CLAUDE.md. Same content; this file is tool-neutral,
---

<!-- Keep in sync with CLAUDE.md. Same content; this file is tool-neutral,
     CLAUDE.md is the Claude Code copy. Edit both when either changes. -->

# AGENTS.md

This file orients any AI coding agent (Codex, Cursor, Aider, Gemini CLI, and
others) working in this repository. Claude Code reads `CLAUDE.md`, which carries
the same guidance. If you change one, change the other.

## What this is

Sinclair is a GPU-accelerated terminal emulator for macOS and Linux, written in
Rust as a Cargo workspace. The GUI is built on [gpui](https://github.com/zed-industries/zed)
(pulled as a git dependency from the zed repo). The GUI is the `app` crate,
whose bin target is `sinclairdev`: a dev build (`cargo run -p app`, debug or
`--release`) is named `sinclairdev` so it never collides with an installed
`sinclair` — it gets its own window title, app id, and single-instance socket and
runs side by side. The release scripts install the same binary as the shipped
`sinclair` command. The app derives this name from its own executable at runtime
(see `crates/app/src/appid.rs`).

## Commands

```sh
cargo run -p app --release        # build and launch the terminal
cargo build --release             # build the workspace
cargo test                        # run all tests (workspace)
cargo test -p vt                  # test one crate
cargo test -p vt screen           # run tests matching "screen" in one crate
cargo clippy --all-targets        # lint

scripts/bundle.sh                 # cargo build --release + assemble dist/Sinclair.app
scripts/dmg.sh                    # package dist/Sinclair.dmg (needs bundle first)
scripts/linux.sh [x86_64|aarch64] # build + package .tar.gz/.deb/.AppImage (Linux)
```

Each crate keeps its tests in a sibling `tests/` directory (e.g.
`crates/vt/tests/`), mirroring the `src/` layout. These are **not** ordinary
integration tests: every crate sets `autotests = false`, and each source file
pulls its test file back in as a private module so unit tests keep access to
private items and the `app` binary can be tested:

```rust
// at the bottom of src/foo.rs
#[cfg(test)]
#[path = "../tests/foo.rs"]
mod tests;
```

Add a new test file the same way (and create the `#[path]` stub in the source
file). Genuine integration tests that exercise only the public API are declared
explicitly as `[[test]]` targets (see `crates/vt` and `crates/terminal`). The
`vt` and `config` crates carry the bulk of the coverage and are pure logic —
prefer adding there.

## gpui dependency

gpui and `gpui_platform` come from a pinned zed git rev. Because cargo
`[patch.crates-io]` entries do not propagate through git dependencies, the root
`Cargo.toml` must mirror zed's own patches (`async-process`, `async-task`).
Requires Rust stable >= 1.96. If you bump the zed rev, re-check zed's root
`Cargo.toml` patch section and update ours to match. See `docs/gpui.md`.

## Architecture

The workspace is layered bottom-up; each crate depends only on those below it.

- **`vt`** — the terminal emulation core. Pure, no I/O: a `vte`-driven parser
  feeds a `Grid`/`Screen` with cursor, modes, scrollback, selection,
  hyperlinks, search, and SGR/charset state. `term/` holds CSI/OSC/DCS dispatch
  and reports. Everything here is testable in isolation.
- **`pty`** — Unix pty allocation and child-process spawn (`rustix`). Unix-only.
- **`terminal`** — runtime glue: `Session::spawn` runs a child on a pty, feeds
  its bytes into a `vt::Terminal` on a reader thread, and emits `Event`s
  (wakeup, title, bell, exit) over an async-capable flume receiver.
- **`cast`** — asciinema v2 `.cast` recording: a `Recorder` writes a header line
  plus timestamped output events as bytes arrive (output only; UTF-8 split
  across reads is carried over). Used by `terminal` for session capture.
- **`container`** — container-backed terminals, pure argv construction with no
  I/O beyond a `$PATH` probe: Docker/Podman detection, the OS profiles behind
  "OS Tabs", and the project **sandbox** — one long-lived container a human and
  a whole agent team share. The sandbox identity-mounts the project (a path
  means the same thing inside and out, so git worktrees stay valid from both
  sides), generates the image `Recipe` that installs the agent CLIs, and
  discovers containers a project already has by label so an editor's
  devcontainer is entered rather than duplicated. `app` drives it; `relay` uses
  the same builders so a `docker exec` assembled on either side cannot drift.
  See `docs/sandbox.md`.
- **`input`** — keyboard/mouse encoding to terminal byte sequences (CSI, kitty
  keyboard protocol, mouse reporting, bracketed paste).
- **`config`** — layered settings: compiled-in defaults overridden by the
  user's `settings.json` (JSON with comments, parsed by `json.rs`), with live
  file watching. `kind.rs` is the typed key schema; bad values become friendly
  diagnostics plus the default and never abort the load. `jsonedit.rs` does
  comment-preserving single-key writes. Default path
  `$XDG_CONFIG_HOME/sinclair/settings.json` or
  `~/.config/sinclair/settings.json`; the legacy `key = value` `config` file
  is still parsed for one-time migration (`app`'s `confwrite::migrate`).
- **`theme`** — 22 built-in color schemes (`builtin/`) plus per-color overrides.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wess/sinclair](https://github.com/wess/sinclair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
