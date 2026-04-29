---
trigger: always_on
description: - MarkLight is a Tauri 2 desktop Markdown editor.
---

# AGENTS.md

## Project snapshot

- MarkLight is a Tauri 2 desktop Markdown editor.
- Frontend lives in `src/` and is built with Vue 3, Pinia, Vite, TypeScript, and Tailwind.
- Desktop/native code lives in `src-tauri/` and exposes filesystem, config, watcher, window, image, and print commands to the frontend.
- The editor core is TipTap/ProseMirror with a custom Markdown parser and serializer in `src/components/Editor/tiptap/markdown/`.

## Essential commands

### Frontend and app

```bash
pnpm install
pnpm dev
pnpm dev:tauri
pnpm build
pnpm build:tauri
pnpm preview
pnpm lint
pnpm lint:fix
pnpm format
```

### Tests

Vitest is configured in `vitest.config.ts`, but there is no `test` script in `package.json`.
Run tests directly:

```bash
pnpm vitest run
```

## What lives where

### Frontend shell

- `src/App.vue` is the orchestration layer. It wires stores, menu events, window events, file operations, file tree state, export actions, focus mode, command palette, and the editor/image/source-mode switch.
- `src/main.ts` is minimal bootstrapping only.

### State and cross-cutting logic

- `src/stores/file.ts` owns the current document path/content/dirty state and last modified timestamp.
- `src/stores/settings.ts` owns app settings, theme selection, config migration, focus mode, and debounced persistence.
- `src/composables/` contains most app behavior that is not editor-schema specific: file open/save, file tree/watchers, export, image preview, menu/window integration.

### Editor

- `src/components/Editor/MarkdownEditor.vue` creates the TipTap editor, registers all custom extensions, debounces serialization back into the file store, emits outline/cursor/word-count updates, and owns editor-local search/replace and image drag-drop handling.
- `src/components/Editor/tiptap/extensions/` holds the custom TipTap extensions. Complex rendering is implemented inline with `addNodeView()` in the extension files rather than split into separate Vue files.
- `src/components/Editor/tiptap/markdown/parser.ts` converts Markdown into a ProseMirror document.
- `src/components/Editor/tiptap/markdown/serializer.ts` converts the ProseMirror document back to Markdown.
- `src/components/Editor/tiptap/markdown/__tests__/roundtrip.spec.ts` is the main automated safety net for Markdown fidelity.

### Themes and export

- `src/themes/` contains the app theme system. Preset themes are JSON files under `src/themes/presets/`.
- `src/utils/wechat-renderer.ts` is a separate Markdown-to-HTML export path for WeChat copy/export. It does not go through the ProseMirror document tree.

### Native layer

- `src-tauri/src/lib.rs` registers commands, menu wiring, file watcher setup, startup file-open handling, and window-close interception.
- `src-tauri/src/commands/fs.rs` contains the file operations exposed to the frontend.
- `src-tauri/src/commands/watch.rs` wraps the shared `notify` watcher.
- `src-tauri/src/commands/config.rs` persists settings to the app config directory as `settings.json`.
- `src-tauri/src/menu.rs` defines native menu items and emits string event ids back to the frontend.

## Architecture and data flow

### Document flow

1. Markdown is loaded from the Rust command layer.
2. `parseMarkdown()` turns it into a ProseMirror document.
3. TipTap renders and edits that document.
4. `serializeMarkdown()` is called on debounced editor updates.
5. The serialized Markdown is written back into the file store and later saved via Rust commands.

### App flow

- `App.vue` treats the editor as one view mode among editor/image/source views.
- Sidebar outline data is emitted upward from `MarkdownEditor.vue`.
- File tree state is independent from current document state and is refreshed from native watcher events.
- Native menu actions are emitted from Rust as string ids and mapped back to handlers in the frontend.

## Non-obvious conventions and gotchas

### Dirty-state handling is intentional

- `src/stores/file.ts` separates `setContent()` from `markUserEdit()`.
- `setContent()` only marks the document dirty after `hasUserEdit` has been set.
- This is important because the editor writes normalized content back into the store programmatically.
- If you change document sync behavior, preserve this distinction or you will reintroduce false dirty states.

### The editor establishes a normalized baseline on load

- In `MarkdownEditor.vue`, after parsing and `setContent()`, the code immediately serializes the ProseMirror document and pushes that baseline back into the file store.
- This avoids dirty-state flicker caused by parser/serializer normalization differences.
- Do not remove this unless you also redesign dirty detection.

### The serializer always normalizes the trailing newline

- `serializeMarkdown()` forces output to end with exactly one trailing newline.
- Round-trip tests normalize expected output the same way.
- If a change appears to add a newline unexpectedly, check the serializer before changing tests.

### Markdown support is split across preprocessors and schema handlers

- `parser.ts` does more than raw `markdown-it` token mapping.
- Frontmatter, callouts, and `$$...$$` math blocks are extracted before `markdown-it` runs and restored from placeholder comments afterward.
- Wikilinks are recognized from plain text tokens.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiaodou997/marklight](https://github.com/xiaodou997/marklight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
