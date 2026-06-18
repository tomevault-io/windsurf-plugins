---
trigger: always_on
description: > This document serves as a navigation guide for AI/automated agents working in this repository. It summarizes the overall architecture of the repository, the responsibilities of each crate in the Cargo workspace, the boundaries of each submodule under the `app/` main binary, and the engineering conventions that must be adhered to before making modifications.
---

# AGENTS.md

> This document serves as a navigation guide for AI/automated agents working in this repository. It summarizes the overall architecture of the repository, the responsibilities of each crate in the Cargo workspace, the boundaries of each submodule under the `app/` main binary, and the engineering conventions that must be adhered to before making modifications.
>
> It is a companion document to `WARP.md`: `WARP.md` is the developer manual (commands, style, processes), while this document is the **code map**. Read `WARP.md` first, then use this document to locate the correct crate/module.

---

## 1. Repository Overview

Warp is a Rust-centric **agentic terminal / development environment**: built on a custom UI framework (WarpUI), it integrates terminal emulation, AI agents, cloud sync (Drive), code review, completion, Notebook, settings, IPC, and other capabilities.

Top-level directories:

| Directory | Purpose |
|------|------|
| `app/` | Main binary crate (`warp`), assembling all subsystems, UI, database migrations, and platform glue layer |
| `crates/` | 67 workspace members, library crates split by responsibility |
| `command-signatures-v2/` | Independent subproject (excluded when running nextest with `--exclude`) |
| `script/` | Cross-platform bootstrap, build, and presubmit scripts |
| `resources/` | Fonts, icons, shell integration scripts, shaders, and other runtime resources |
| `docker/` | Containerized build configurations |
| `specs/` | Product/Technical spec documents |
| `.agents/skills`, `.claude/skills` | Skill descriptions for agent workflows (PR creation, error fixing, feature gating/rollout, etc.) |
| `.warp/`, `.config/`, `.cargo/`, `.vscode/` | Configurations for various tools |

Build System: Cargo workspace, `resolver = "2"`. `default-members` is intentionally restricted to the subset that is frequently compiled/tested (see `Cargo.toml`). `serve-wasm` and `integration` are excluded from `default-members` by default.

License Split:
- `crates/warpui` and `crates/warpui_core` → MIT
- Others → AGPL-3.0-only

---

## 2. Top-Level Architectural Layers

There are roughly 4 layers from bottom to top. When adding new code or locating a bug, first determine which layer the change belongs to, and **never introduce circular or downward-pointing dependencies across layers**.

```
app/  (Main binary: assembly, entry points, platform glue, persistence migrations, UI view root)
  ↑
Product domain crates: ai / computer_use / vim / onboarding /
                      warp_completer / lsp / languages / code-review …
  ↑
Framework crates: warpui / warpui_core / warpui_extras / editor /
            ui_components / sum_tree / syntax_tree
  ↑
Infrastructure crates: warp_core / warp_util / http_client /
                websocket / ipc / jsonrpc / persistence / graphql /
                managed_secrets / virtual_fs / watcher / asset_cache …
```

Key Architectural Patterns (see `WARP.md` for details):

1. **Entity-Handle System**: `App` globally owns all view/model entities. Views reference each other via `ViewHandle<T>` rather than direct ownership.
2. **Element / Action**: The UI is composed of a declarative Element tree + Action event system (Flutter-style).
3. **Cross-Platform**: Native implementation for macOS / Windows / Linux + WASM target. Platform-specific code is isolated using `#[cfg(...)]`.
4. **AI Integration**: Agent Mode and context index. Code is concentrated in `app/src/ai` (389 files) and `crates/ai`.
5. **Cloud Sync**: `Drive` enables object synchronization across multiple devices. See `app/src/drive` and `crates/warp_files`.
6. **Feature Flag**: Runtime rollout/gating takes precedence over `#[cfg]`. The enum is defined in `crates/warp_core/src/features.rs`.

---

## 3. `crates/` Overview

The table below lists all 67 crates grouped by topic. Each row contains only a **one-sentence description of its responsibility**; to view implementation details, open `crates/<name>/src/lib.rs` directly (many crates have `//!` module documentation at the top of `lib.rs`).

### 3.1 UI Framework / View Layer

| Crate | Responsibility |
|-------|------|
| `warpui_core` | WarpUI framework core (MIT): `App` / `Entity` / `ViewHandle` / `AppContext` and other infrastructure |
| `warpui` | WarpUI high-level components, Element tree, layout, rendering pipeline (MIT) |
| `warpui_extras` | Optional extensions of WarpUI, not all features are enabled by default |
| `ui_components` | High-level component library reused across views (buttons, inputs, lists, modals, etc.) |
| `editor` (`warp_editor`) | Text editor: buffers, selections, cursors, keymaps, undo stack |
| `sum_tree` | Persistent balanced B-tree, core data structure for editor / Notebook / large lists |
| `syntax_tree` | Tree-sitter wrapper and syntax highlighting support |
| `markdown_parser` | Markdown parsing (used for AI messages, document views, Notebooks, etc.) |
| `vim` | Vim mode keybindings and operational semantics |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codeitlikemiley/waz](https://github.com/codeitlikemiley/waz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
