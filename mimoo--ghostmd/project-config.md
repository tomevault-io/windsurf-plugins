---
trigger: always_on
description: GhostMD is a native macOS note-taking app written in Rust using GPUI (GPU-accelerated UI framework). Notes are plain `.md` files in `~/Documents/ghostmd/`.
---

# GhostMD — Agent Guide

## Project Overview

GhostMD is a native macOS note-taking app written in Rust using GPUI (GPU-accelerated UI framework). Notes are plain `.md` files in `~/Documents/ghostmd/`.

## Architecture

Cargo workspace with two crates:

- **`crates/ghostmd-core/`** — Pure business logic, zero UI dependencies. All data structures and algorithms live here.
  - `buffer.rs` — Rope-based text buffer (ropey 2.0.0-beta.1) with branching undo tree (undo 0.44)
  - `note.rs` — Note CRUD and auto-save
  - `diary.rs` — Date-based diary path generation (`diary/YYYY/month-name/DD/`)
  - `tree.rs` — File tree model with recursive scan
  - `search.rs` — Fuzzy file search (nucleo-matcher) + full-text content search (grep-searcher)
  - `path_utils.rs` — Collision-safe path generation (`unique_path()` appends `-2`, `-3`, ...)

- **`crates/ghostmd/`** — Native GPUI application shell. State machines are tested independently of GPUI rendering.
  - `app_view/` — **Main GPUI view** (refactored into submodules):
    - `mod.rs` — `GhostAppView` struct, constructor, `Render` impl with action handlers
    - `workspace.rs` — Workspace CRUD, pane focus, split management, tab switching
    - `file_ops.rs` — File open/create/move/trash, location picker, `update_editor_paths()` helper, update mechanism
    - `rendering.rs` — All render methods: tab bar, split nodes, overlays (file finder, agentic search, location picker, command palette), context menu
    - `overlays.rs` — Open/close methods for all overlays
    - `palette_dispatch.rs` — Command palette command list and dispatch, rename mode
    - `ai_commands.rs` — AI rename tab/file, suggest folder, agentic search, search matches
    - `session.rs` — Session persistence (save/restore workspaces to JSON)
    - `split_node.rs` — `SplitNode` binary tree with directional navigation
    - `fs_watcher.rs` — File system watcher for external changes (notify crate)
  - `editor_view.rs` — GPUI view wrapping `InputState` for editing a single note. Tracks path, dirty flag, auto-save timing
  - `file_tree_view.rs` — GPUI view for the sidebar file tree (renders `FileTreePanel`)
  - `app.rs` — Legacy root state machine. `#![allow(dead_code)]` — fully superseded by `GhostAppView`'s direct `root` and `sidebar_visible` fields
  - `editor.rs` — Legacy editor state machine. `#[cfg(test)]` only
  - `file_tree.rs` — File tree sidebar state machine with keyboard navigation
  - `search.rs` — File finder state machine (wired to GPUI via `app_view/rendering.rs`)
  - `tabs.rs` — Legacy tab manager. `#[cfg(test)]` only
  - `splits.rs` — Legacy flat split pane layout. `#[cfg(test)]` only
  - `palette.rs` — Command palette state machine (filtering, selection)
  - `ai.rs` — AI manager for suggestion storage/retrieval. `#[cfg(test)]` only
  - `theme.rs` — Multi-theme support (`GhostTheme`, `ResolvedTheme` pre-converted HSLA cache) with `rgb_to_hsla` converter
  - `keybindings.rs` — GPUI action definitions and keyboard shortcut registration
  - `assets.rs` — Asset loading (fonts)
  - `main.rs` — Application entry point, window creation

## Building & Testing

```sh
cargo build                     # full build
cargo test                      # all tests
cargo test -p ghostmd-core      # core logic only
cargo test -p ghostmd           # UI state machine tests
cargo bench -p ghostmd-core     # criterion benchmarks (buffer, tree, search)
cargo clippy --tests            # must pass with zero warnings
```

Requires Rust 1.75+ and Xcode with Metal Toolchain on macOS.

## Versioning & Releases

- Version is defined in `crates/ghostmd/Cargo.toml` — this is the single source of truth.
- CI auto-creates a git tag (`vX.Y.Z`) when the version changes on main (`.github/workflows/auto-tag.yml`).
- The release workflow (`.github/workflows/release.yml`) triggers on `v*` tags and builds macOS binaries.
- **To release**: bump the version in `Cargo.toml` and push to main. CI handles the rest.
- **Known CI caveat**: Tags created by `GITHUB_TOKEN` (e.g. via the auto-tag workflow) do NOT trigger other workflows. If the release workflow doesn't fire after auto-tag, delete the remote tag and re-push it locally: `git push origin :refs/tags/vX.Y.Z && git tag vX.Y.Z && git push origin vX.Y.Z`.

## Key Technical Details

- **ropey 2.0.0-beta.1** uses byte indices, not char indices. All buffer operations work in bytes.
- **undo 0.44** uses `Action` trait (not `Edit`). Methods: `apply`, `undo`, `merge` returning `Merged`.
- **GPUI dependencies** require pinning `core-foundation = "=0.10.0"` and `core-text = "=21.0.0"` to avoid conflicts.
- **Diary paths** use lowercase month names: `diary/2026/march/03/HHMMSS-slug.md`.
- **String truncation** must use `chars().take(n)` not byte slicing `&s[..n]` — byte slicing panics on multi-byte UTF-8.
- **Dead code policy**: No crate-level `#![allow(dead_code)]`. Each module/item that is tested but not yet wired to GPUI gets its own `#[allow(dead_code)]` or module-level `#![allow(dead_code)]`. When wiring new features, remove the corresponding allows.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mimoo/ghostmd](https://github.com/mimoo/ghostmd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
