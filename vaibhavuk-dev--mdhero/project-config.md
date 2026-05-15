---
trigger: always_on
description: pnpm tauri dev          # Run dev server with hot-reload (frontend + Rust)
---

# CLAUDE.md

## Commands

```bash
pnpm tauri dev          # Run dev server with hot-reload (frontend + Rust)
pnpm tauri build        # Production build (outputs DMG/MSI)
pnpm build              # Build frontend only (SvelteKit static)
pnpm check              # TypeScript type checking
cargo build --manifest-path src-tauri/Cargo.toml  # Build Rust backend only
```

Port 1420 is used for dev. Kill stale processes with `lsof -ti:1420 | xargs kill -9` before relaunching.

## What This Is

A lightweight, native desktop Markdown viewer & editor (~11MB) built with Tauri v2 + SvelteKit. Opens `.md` files and renders them beautifully with syntax highlighting, math, and diagrams. Includes a toggleable in-app editor (`Cmd+E` to enter, `Cmd+S` to save), an LLM paste-to-render mode for AI-generated markdown, and an Open URL fetcher. Targets macOS and Windows.

## Architecture

Single-window app. Rust backend handles file I/O, write-back, and watching; SvelteKit frontend handles rendering, editing, and UI.

**Rendering pipeline** (`src/lib/renderer/pipeline.ts`): markdown-it parses markdown → highlight.js highlights code (25+ languages, synchronous, no WASM) → markdown-it-texmath renders KaTeX math → DOMPurify sanitizes HTML → Svelte renders via `{@html}`. Mermaid diagrams are post-processed after DOM update in `MarkdownRenderer.svelte`. A custom markdown-it core rule stamps `data-source-line="N"` on top-level block tokens — used for cross-mode scroll sync.

**File watching** (`src-tauri/src/watcher.rs`): Watches the parent directory (not the file directly) using the `notify` crate with 500ms debounce. This survives atomic saves (delete + rename) that editors like VS Code and vim use. Emits `file-changed` Tauri event filtered by target filename. The frontend watcher (`src/lib/tauri/watcher.ts`) suppresses reloads within 1.5s of an own-write (after `Cmd+S` from the in-app editor) by checking `tab.lastSavedAt`.

**Tab + edit state** (`src/lib/stores/tabs.ts`): Each tab carries `content` (on-disk), `editContent` (in-progress), `dirty`, `isEditing`, and `lastSavedAt`. The `Tab.isEditing` field is per-tab so users can edit one file while reading another. Switching tabs preserves edit state. External file changes (after the suppression window) update `content` only and recompute `dirty` — never clobber `editContent` while editing.

**Scroll sync** (`src/lib/utils/scroll-sync.ts`): Toggling between viewer/raw/editor preserves the source-line position. Viewer reads the topmost visible block's `data-source-line`. Raw and editor estimate the line via `scrollTop / lineHeight`. The destination scrolls to put the same source line at the top. Mode toggles route through `switchMode(target)` in `+page.svelte`.

**State management**: Svelte stores in `src/lib/stores/`. `tabs.ts` (multi-tab + edit state), `document.ts` (active document), `theme.ts` (light/dark/system, class-based), `settings.ts` (font/line/width/family, persisted to localStorage), `pinned.ts` (pinned folders), `recents.ts` (recent files), `toc.ts` (TOC entries + active heading), `updater.ts` (in-app update check against GitHub Releases).

**Tauri bridge** (`src/lib/tauri/`): `files.ts` wraps IPC commands and coordinates opening/reloading/saving. `watcher.ts` listens for `file-changed` events with own-save suppression.

## Key Files

### Frontend
- `src/lib/renderer/pipeline.ts` — Core rendering: markdown-it + highlight.js + KaTeX + DOMPurify + source-line stamps
- `src/lib/components/MarkdownRenderer.svelte` — Renders HTML, handles Mermaid post-processing, TOC extraction via IntersectionObserver
- `src/lib/components/Editor.svelte` — In-app editor (textarea, fixed positioning, monospace, Tab inserts spaces)
- `src/lib/components/PasteModal.svelte` — Paste-to-render + Open URL modal
- `src/lib/components/SearchOverlay.svelte` — Cmd+F search with mark.js highlighting
- `src/lib/components/TabBar.svelte` — Tabs with home tab, drag reorder, dirty `•` indicator
- `src/lib/components/Toolbar.svelte` — All toolbar buttons; everything stays visible and disables (with tooltip) when not applicable
- `src/lib/components/EmptyState.svelte` — Home tab UI: pinned folders, recent files, plans
- `src/lib/components/OpenDialog.svelte` — Cmd+O dialog: browse, recent, plans, pinned folders, URL
- `src/lib/utils/scroll-sync.ts` — Cross-mode scroll anchoring on source line
- `src/lib/utils/llm.ts` — LLM output detection and unescaping (`\n`, `\"`, JSON strings)
- `src/lib/utils/url.ts` — URL → raw URL conversion (GitHub blob, gist, GitLab, Bitbucket)
- `src/lib/utils/clipboard.ts` — Copy as rich text (HTML) or raw markdown
- `src/lib/tauri/files.ts` — File open, save, reload (dialog, drag-drop, CLI, "Open With")
- `src/lib/tauri/watcher.ts` — Frontend file-changed handler with own-save suppression
- `src/routes/+page.svelte` — Main page, keyboard shortcuts, mode switching, render gating

### Backend
- `src-tauri/src/lib.rs` — Tauri setup: plugins, commands, menu, state, `OpenedFiles` buffer for "Open With"
- `src-tauri/src/commands.rs` — Rust IPC commands: `read_markdown_file`, `write_markdown_file`, `list_claude_plans`, `list_folder_md_files`
- `src-tauri/src/watcher.rs` — Rust file watcher (notify crate, watches parent dir)
- `src-tauri/src/menu.rs` — Native menu bar

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vaibhavuk-dev/mdhero](https://github.com/vaibhavuk-dev/mdhero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
