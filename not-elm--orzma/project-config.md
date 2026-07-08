---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture

orzma is a terminal multiplexer that runs as a single native GUI application. It is a hybrid Rust + TypeScript codebase organized as one Cargo workspace and one pnpm workspace sharing the same tree. There is no daemon, no HTTP server, and no browser-side frontend — terminal emulation, GPU rendering, layout, input, and webview rendering all run in one Bevy ECS world.

### Rust workspace (`Cargo.toml`)

The workspace root package is the one and only binary; library crates live under `crates/`. Edition 2024, toolchain pinned to `1.95` (`rust-toolchain.toml`).

- `orzma` (workspace root, `src/main.rs`) — the single binary: a Bevy 0.18 app. `main()` builds one `App` and adds `DefaultPlugins` (configured with a `WindowPlugin` titled "orzma") plus `cef_plugin(orzma_registry.clone(), cef_profile.path())` (from `bevy_cef`), then `AppModePlugin`, then the orzma plugins:
  - `SurfacePlugin`, `DefaultSessionPlugin`, `ClipboardPlugin`, `TerminalHandlePlugin` (from `orzma_tty_engine`), `TerminalRendererPlugin` (from `orzma_tty_renderer`), `RenderPlugin` (`render::tmux`'s tmux grid/paint pipeline), `TmuxLifecyclePlugin` (`session::tmux`; wraps `TmuxSessionPlugin` from `orzma_tmux`, the sole multiplexer, plus `AdoptPlugin`, `TmuxLocalePlugin`, `WebviewTokensPlugin`), `ActionPlugin`, `OrzmaConfigsPlugin`, `FontBridgePlugin`, `OrzmaBootstrapPlugin`, `OrzmaInputPlugin` (`input`'s root plugin, aggregating `ShortcutsPlugin`, `OptionAsAltPlugin`, `KeyboardInputPlugin`, `MouseInputPlugin`, the tmux-mode and default-mode input dispatchers), `OrzmaUiPlugin` (`ui`'s root plugin, aggregating the UI root plus the tmux-mode and default-mode UI subtrees);
  - `OrzmaWebviewPlugin` (from `orzma_webview`), `CopyModePlugin`, `CopyModeIndicatorPlugin`, `CopyPromptPlugin`, `WindowTitlePlugin`;
  - the input plugins `FocusSyncPlugin`, `HyperlinkInputPlugin`, `ImePlugin`, and `ImeOverlayPlugin`.
  - The in-process webview feature — CEF render wiring, the control-socket listener, the `window.orzma` back-channel, OSC 5379 `mount` / `unmount`, and webviews — is aggregated under `OrzmaWebviewPlugin` (from `crates/orzma_webview`).

  The root `Cargo.toml` depends on `orzma_webview` (path dep, `features = ["cef"]`) and on `bevy_cef` (git dep, `branch = "passthrough"`, `features = ["debug"]`). A root `[features] debug` flag enables the CEF `remote-debugging-port` (a local Chromium DevTools / CDP endpoint on `127.0.0.1:9222`) for inspecting the embedded webview; it is off by default (`cargo run --features debug`).

- `crates/orzma_tty_engine` (`orzma_tty_engine`) — Bevy-native terminal: PTY ownership and `alacritty_terminal` VT emulation, emitting coalesced `FrameSnapshot` / `FrameDelta` against the `orzma_tty_renderer` schema. Exposes `TerminalHandlePlugin`.
- `crates/orzma_tty_renderer` (`orzma_tty_renderer`) — GPU terminal renderer plus the grid schema shared with `orzma_tty_engine`. `TerminalRendererPlugin` wires the grid, material, and glyph sub-plugins (`TerminalGridPlugin`, `TerminalMaterialPlugin`, `TerminalGlyphPlugin`) and hyperlink-hover state; `schema` holds the cell/grid types both crates render against.
- `crates/orzma_webview` (`orzma_webview`) — the in-process webview feature: depends on `orzma_tty_engine`, `orzma_webview_host`, and (behind the `cef` feature) `bevy_cef`. Aggregates CEF render wiring, the OSC 5379 `mount` / `unmount` handler, the `window.orzma` back-channel, the control-socket listener that mints Tier 1 dynamic webview handles, and focus management. Exposes `OrzmaWebviewPlugin` and `cef_plugin`.
- `crates/webview_host` (`orzma_webview_host`) — Tokio-free webview host integration for orzma: a per-handle `RuntimeRoot` runtime directory tree (the 0700 socket dir the control plane mints), and (behind the `cef` feature) serving dynamically-registered Tier 1 webview assets from disk/memory through a `bevy_cef` `orzma://` custom scheme via the `bevy_cef_core` path dep. The `cef` feature is off by default so the core builds/tests with std only. Exposes `WebviewAsset`, `WebviewAssetRegistry`, `custom_orzma_scheme`, and `RuntimeRoot`.
- `crates/orzma_tmux` (`orzma_tmux`) — the sole multiplexer: owns a `tmux -CC` control-mode connection, drains its transport events into the Bevy world, tracks connection lifecycle, and projects tmux session/window/pane state as ECS entities (`TmuxSession` / `TmuxWindow` / `TmuxPane`). Rendering lives in `src/` (the tmux render/input/UI plugins), not here. Exposes `TmuxSessionPlugin`. (Built on `crates/tmux_control` and `crates/tmux_control_parser`, the sans-io tmux control-mode client and parser.)
- `crates/orzma_configs` (`orzma_configs`) — config loader. Reads `~/.config/orzma/config.toml` (or `$ORZMA_CONFIG` / `$XDG_CONFIG_HOME` overrides) and resolves it against built-in defaults.

In-process webview rendering is provided by the external `bevy_cef` crate (a git dependency on the `passthrough` branch, CEF v145 pinned to `145.6.1+145.0.28` in the justfile). Both the renderer and the helper render process come from `bevy_cef` / `export-cef-dir`; see `just setup-cef`.

### TypeScript workspace (`pnpm-workspace.yaml`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [not-elm/orzma](https://github.com/not-elm/orzma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
