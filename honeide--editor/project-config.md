---
trigger: always_on
description: High-performance, cross-platform code editor surface published as `@honeide/editor`. Designed to be embeddable by other developers for markdown editors, config editors, query consoles, etc. Compiled to native binaries via **Perry** (TypeScript-to-native compiler, v0.5.1235).
---

# hone-editor

## Project Overview
High-performance, cross-platform code editor surface published as `@honeide/editor`. Designed to be embeddable by other developers for markdown editors, config editors, query consoles, etc. Compiled to native binaries via **Perry** (TypeScript-to-native compiler, v0.5.1235).

**Key constraint**: Perry's Canvas widget has no text-on-canvas capability. The editor uses custom Rust FFI crates per platform for native text rendering (Core Text on macOS/iOS, DirectWrite on Windows, Pango/Cairo on Linux, Skia on Android, DOM on Web).

## Tech Stack
- **Core logic**: TypeScript (platform-independent, shared across all 6 targets)
- **Native rendering**: Rust FFI crates (one per platform)
- **Syntax highlighting**: Lezer parser ecosystem (@lezer/*)
- **Build**: Perry compiler (`perry compile`)
- **Test runner**: `bun test`
- **Package manager**: Bun

## Architecture

```
core/           Platform-independent TypeScript
  buffer/       Piece table + B-tree rope text buffer (O(log n) ops)
  document/     EditorDocument, EditBuilder, encoding detection
  cursor/       Multi-cursor management, selections, word boundaries
  commands/     Command registry + editing/navigation/selection/clipboard/multicursor
  history/      Undo/redo with time-based coalescing
  viewport/     Virtual scrolling, line height cache, scroll controller
  tokenizer/    Lezer syntax highlighting for 10 languages
  search/       Literal + regex search/replace, incremental search
  folding/      Indent-based + syntax-based code folding
  diff/         Myers diff algorithm, inline char-level diff, hunk operations
  lsp-client/   LSP client: JSON-RPC transport, protocol types, capability negotiation
  dap-client/   DAP client: debug session lifecycle, breakpoints, stepping, variables
view-model/     Reactive state bridging core → rendering
  editor-view-model.ts   Central orchestrator, key bindings, mouse handling
  theme.ts               Dark + light themes, token color mappings
  line-layout.ts         RenderedLine computation
  cursor-state.ts        Blink controller, IME composition
  gutter.ts              Line numbers, fold indicators, breakpoints, diff markers
  find-widget.ts         Find/replace widget controller
  ghost-text.ts          AI ghost text inline completion
  minimap.ts             Minimap data generation
  overlays.ts            Autocomplete, hover, parameter hints, diagnostics overlays
  decorations.ts         Search highlights, selection, diagnostic underlines
  diff-view-model.ts     Side-by-side/inline diff view state
native/         Platform-specific rendering bridge + Rust FFI crates
  ffi-bridge.ts          NativeEditorFFI interface, NoOpFFI test impl
  render-coordinator.ts  Bridges EditorViewModel → FFI calls with dirty tracking
  touch-input.ts         Touch gesture handler (tap, pan, pinch, long press)
  word-wrap.ts           Word wrap computation + WrapCache
  index.ts               Barrel export
  macos/        Core Text + Core Animation + Metal (Rust)
  ios/          Core Text + UIKit (Rust, shares rendering with macOS)
  windows/      DirectWrite + Direct2D + DirectComposition (Rust)
  linux/        Pango + Cairo + X11/Wayland (Rust)
  android/      Canvas + Skia via JNI (Rust)
  web/          DOM spans + CSS + WASM (Rust)
tests/          Unit and integration tests
```

## Key Design Decisions
- **Text buffer**: Piece table with B-tree rope indexing — O(log n) for all offset/line operations
- **Line endings**: Always normalized to `\n` internally; original line ending style preserved in EditorDocument for saving
- **No external editor dependencies**: No CodeMirror, Monaco, prosemirror. Self-contained.
- **Edits are atomic**: EditBuilder collects edits, applies via `buffer.applyEdits()` in reverse offset order
- **Undo coalescing**: Single-character inserts/deletes within 500ms are grouped; newlines always start new groups
- **Multi-cursor**: Cursors sorted by position, merged when overlapping, desired column preserved across vertical movement
- **Virtual scrolling**: Only visible lines + 10-line buffer zone are rendered

## Cross-Platform Strategy
- `core/` and `view-model/` are 100% shared across all platforms
- macOS and iOS share Core Text rendering code
- Only `native/` Rust FFI crates are platform-specific
- FFI contract is identical across all platforms (same function signatures)
- **HarmonyOS** (`native/harmonyos/`) is currently a no-op stub: every
  `hone_editor_*` symbol link-resolves so Perry-compiled apps embedding
  the editor load on the device, but the surface renders blank. Real
  ArkTS-side rendering (canvas + `@ohos.pasteboard` + IME) is a follow-up.

## Perry AOT Codegen Notes
Perry v0.4.14+ compiles standard TypeScript patterns correctly, including: `?.`, `??`, `for...of`,
ES6 shorthand `{ key }`, `array.push/indexOf/map` on class fields, `obj[variable]` dynamic keys,
`/regex/.test()`, character range comparisons, and `Array.sort()` with comparators (TimSort).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HoneIDE/editor](https://github.com/HoneIDE/editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
