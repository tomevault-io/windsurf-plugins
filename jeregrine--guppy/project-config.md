---
trigger: always_on
description: Guppy is an Elixir UI framework that renders through GPUI using a NIF-backed native runtime.
---

# Guppy

## What this repo is

Guppy is an Elixir UI framework that renders through GPUI using a NIF-backed native runtime.

The intended architecture is:

- Elixir processes own UI state
- Elixir renders that state into a simple IR tree
- native code turns that IR into GPUI elements
- GPUI handles layout, paint, focus, scrolling, and windows
- native events roundtrip back to the owning Elixir process

This project is still unreleased. Do **not** preserve backwards compatibility just because some older internal shape existed.

## Repository scope

This `AGENTS.md` applies to the `./guppy` repo only.

Important repo rules:

- do **not** keep compatibility shims just because they already exist
- if a current design is in the way, replace it cleanly
- optimize for architectural clarity and correctness
- the jj/git repo root is `./guppy`
- do **not** initialize or commit from the parent directory unless explicitly asked
- use `jj` from inside `./guppy`

## Current architecture

Current high-level flow:

1. Elixir builds IR and calls the public API in `lib/guppy.ex`
2. `Guppy.Server` owns view ids, ownership, and event routing
3. `Guppy.Native.Nif` dispatches directly into NIF entrypoints
4. Rustler handles NIF bootstrap, exports, and BEAM interop
5. Rust decodes ETF into native IR
6. Rust enqueues main-thread requests directly into the GPUI runtime queue
7. `BridgeView` renders IR into GPUI elements
8. native events go back through Rustler into the BEAM
9. `Guppy.Server` forwards them to the owning Elixir process

Important current invariants:

- Elixir is the source of truth for UI state
- rendering is full-tree replacement from Elixir's point of view
- retained native state must be keyed by stable identity and pruned aggressively
- explicit node ids win over generated path ids
- style-op lists are ordered and order must be preserved
- IR validation should reject unknown node keys; if a key is allowed it should be validated, decoded, and rendered or deliberately documented
- prevalidated IR wrappers may skip Elixir-side validation, but must unwrap before native decode
- native/main-thread requests carry deadlines; stale queued requests must not mutate native state after caller timeout
- `window_close_requested` is informational today, not a synchronous veto protocol

## Important current implementation details

### Elixir side

- `Guppy.Server` is the central runtime server
- there is **not** a forwarding NIF GenServer anymore
- `Guppy.Native.Nif` is now a direct Elixir wrapper module around the NIF functions
- `Guppy.Window` is the preferred assign-based per-window process abstraction
- `Guppy.Component` / `~GUI` is the preferred template authoring path
- `Guppy.Markdown` is an Elixir-side Markdown-to-IR component for a small subset; do not add Zed markdown crates unless explicitly designing that dependency
- `Guppy.IR.validated/1` and `Guppy.IR.validated!/1` wrap trusted/static IR after one validation pass; server APIs unwrap before native dispatch
- `Guppy.Window` monitors the Guppy runtime server and reopens from current assigns after supervised server restart; while reopen retry has `view_id: nil`, rerenders are skipped/deferred instead of rendering to an unknown view
- runtime telemetry events exist for native NIF calls (`[:guppy, :native, :nif]`), server-mediated native requests (`[:guppy, :native, :request]`), native event routing (`[:guppy, :event, :route]`), and `Guppy.Window` rerenders (`[:guppy, :window, :rerender]`)
- native root views bind Tab/Shift-Tab for GPUI tab-stop traversal, track keyboard-vs-mouse focus-visible state for `focus_visible_style`, and emit app/window activation and window move/resize lifecycle events from GPUI window observers
- file dialogs support cancellation, default directories/names, extension allow-list filters, and logical `owner_view_id` liveness/ownership checks; GPUI 0.2.2 does not expose sheet-style owner-window APIs
- div-like nodes support a narrow native opacity animation spec keyed by stable animation id
- app/runtime menus, Dock menus, and app badges are process-owned via `Guppy.set_menus/1` / `Guppy.set_menus/2`, `Guppy.set_dock_menu/1` / `Guppy.set_dock_menu/2`, and `Guppy.set_app_badge/1` / `Guppy.set_app_badge/2`; callback actions route back through `Guppy.Server` and process-owned shell state is cleared when the owner exits
- `use Guppy.App` modules own app menus, Dock menus, and app badges from config/runtime setters, and handle app lifecycle events such as `"app_activated"` / `"app_deactivated"` with optional `handle_event/3`
- app command helpers include `Guppy.App.command_bindings/1`, `Guppy.App.open_command_palette/1`, `Guppy.App.open_context_menu/3`, and `Guppy.App.focus_window/2` for command-backed shortcuts, transient overlays with focus return, and app-window activation
- semantic `data_table` and `tree` primitives are first-pass virtualized/list-backed nodes; Elixir owns sort, selection, expansion, and context-menu state
- overlay semantics are documented in `docs/overlays.md`: Elixir owns open/close state, select/popover support keyboard close/navigation, select/popover positioning is typed, and nested native overlays inside popovers are rejected until a full overlay stack model exists

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeregrine/guppy](https://github.com/jeregrine/guppy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
