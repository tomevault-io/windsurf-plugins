---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
cargo build --workspace              # build every crate
cargo test --workspace               # run all tests
cargo test -p tili-tree              # test a single crate (the only one that runs on non-macOS)
cargo test -p tili-tree <test_name>  # run a single test
cargo run --bin tili-daemon          # run the daemon directly (not via `cargo install`)
cargo run --bin tili -- ping         # run the CLI directly
```

Before committing, run the exact gate CI enforces (a red PR blocks merge):

```sh
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```

If `cargo fmt --check` fails, run `cargo fmt` and re-stage. Clippy warnings
are hard errors (`-D warnings`); don't `#[allow]` one without a one-line
comment explaining why (see the `#[allow(dead_code)]` on `Tree` in
`tili-tree` for the pattern).

`tili-ax` (and anything depending on it) only builds on macOS — it links
against `AXUIElement`/Core Graphics/Core Foundation, and needs full Xcode
(not just CLT; see CONTRIBUTING.md). `tili-tree` has zero macOS
dependencies by design; prefer adding logic there over `tili-ax` when
possible so it stays testable without a Mac.

## Comments

A comment stays scoped to the function/logic it sits next to: why *this*
code is written the way it is, not the history of how it got there. Don't
reference other bugs, session narrative ("this was reverted because...",
"an earlier attempt tried..."), other tools/projects, or issue-tracker-style
context — that belongs in a commit message or PR description, not in the
source. If a comment needs a sentence about a rejected alternative, phrase
it as a property of *this* code ("not X, because Y" is fine; "we used to do
X but changed it after Z happened" is not).

## Architecture

This is a Cargo workspace; the crate split and one-way dependency
direction are hard boundaries. **The full per-crate design notes — with
the hardware-confirmed findings and history behind every rule below —
live in one file per crate under
[docs/architecture/](docs/ARCHITECTURE.md). Read the relevant crate's
file there before changing event flow, window classification, parking,
focus sync, polling/timing, multi-monitor handling, or release signing.**

- **`tili-tree`** — the container tree and layout algorithms (Tiles/BSP,
  Accordion). No `AXUIElement`, no CoreFoundation, no `unsafe`; operates
  on plain `Rect`/`WindowId` so it's fully unit-testable without macOS.
  Callers use `focus_in_direction` (Accordion-aware), not plain `navigate`.
- **`tili-ax`** — the only crate allowed to touch the Accessibility API;
  depends on `tili-tree` only for geometry types. `src/window.rs` owns the
  single private API call in the codebase (`_AXUIElementGetWindow`) plus
  window classification (`WindowKind`); `src/frame_setter.rs` defines
  `WindowFrameSetter` — the seam every real frame write goes through.
  Each OS event source (`workspace.rs`, `watch.rs`, `display.rs`,
  `hotkey.rs`, `mouse.rs`) runs its own dedicated `CFRunLoop` thread and
  only sends messages.
- **`tili-config`** — KDL parsing/validation into `Config`, plus
  file-watch hot-reload. Runtime-agnostic (`std::sync::mpsc`, not tokio);
  no cross-section semantic validation (that's `tili-daemon`'s job).
  **KDL v2 booleans are `#true`/`#false`** — bare `true`/`false` is a
  parse error; easy to get wrong in test fixtures and example configs.
- **`tili-ipc`** — `Command`/`Response` types shared by daemon and CLI,
  plus socket path/framing. Protocol changes belong here, never duplicated
  in both binaries. `parse.rs` is infallible by design — unknown command
  strings become `Command::Raw` and fail at `dispatch()` time, so a typo'd
  config still loads.
- **`tili-daemon`** — the window manager process. `state.rs` holds
  `WmState`: live `AxWindow` handles, one `Tree` per workspace holding both
  tiled and floating windows (a floating window is a `Node::Floating` leaf
  — addressable via `workspace_focus` like any tiled one, but excluded from
  layout sizing), and a `placements` index for O(1) window→workspace
  lookup plus disposition. `dispatch.rs` has the single
  `dispatch(&mut WmState, Command) -> Response` both the socket and hotkey
  paths call (`Command::Shutdown` is the one documented exception — it's
  process lifecycle, handled in `main.rs`'s loop). `dispatch()` syncs
  focus from real macOS frontmost state synchronously before every
  command — deliberately not a reactive background sync (race-prone;
  see docs/architecture/tili-daemon.md). `main.rs` is one `tokio::select!` loop; no
  locks around `WmState`, only one branch touches it at a time.
- **`tili-cli`** — thin socket client; the binary is named `tili`. No
  business logic here — new behavior belongs in `tili-daemon` behind a
  `Command`. Two documented exceptions: `tili start`/`stop` (LaunchAgent
  management, filesystem-only) and `tili status`'s custom wording.
- **`tili-menubar`** — `NSStatusItem` workspace badge; stays in sync via a
  server-side long-poll (`Command::WaitForChange`), not polling. Its

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itsdezen/tili](https://github.com/itsdezen/tili) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
