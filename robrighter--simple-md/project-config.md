---
trigger: always_on
description: Local-first desktop Markdown editor. Tauri 2 + Rust backend, React 19 + TypeScript + Vite frontend. macOS is the primary target today (Windows/Linux are in scope per the spec but unverified).
---

# Simple MD

Local-first desktop Markdown editor. Tauri 2 + Rust backend, React 19 + TypeScript + Vite frontend. macOS is the primary target today (Windows/Linux are in scope per the spec but unverified).

The filesystem is the source of truth — no proprietary store, no cloud, no DB.

Detailed product/tech spec: `docs/product-and-technical-spec.md`.

## Commands

```bash
npm install
npm run dev          # Vite only (browser at :1420, no Tauri shell)
npm run lint         # eslint flat config
npm run build        # tsc -b && vite build (writes ./dist)
npm run tauri:dev    # full desktop app, hot reload
npm run tauri:build  # release bundle (DMG packaging is not yet polished)
npm run tauri build -- --debug --bundles app   # debug .app bundle
```

Debug app bundle output: `src-tauri/target/debug/bundle/macos/Simple MD.app`.

## Layout

```
src/
  App.tsx                  Top-level shell: tabs, workspaces, modes, file ops, recents
  main.tsx                 React entry
  app/
    types.ts               DocumentMode, TreeNode, OpenDocument, ChartSpec, etc.
    sampleDocument.ts      Welcome doc + sidebar hint
  components/              Pure presentational shell pieces (Toolbar, TabStrip, StatusBar, ModeToggle, WorkspaceSidebar)
  features/
    editor/EditorPane.tsx  CodeMirror 6 source editor
    preview/MarkdownPreview.tsx  react-markdown + remark-gfm + remark-math + rehype-katex + rehype-highlight; intercepts ```chart fences
    charts/InlineChart.tsx Recharts renderer for chart fences
    reports/               reportRegistry + TaskReport (structured report descriptors attached to docs)
  lib/
    desktop.ts             Thin wrappers around Tauri invoke + dialog/opener plugins; gracefully no-ops in browser via isTauri()
    document.ts            Path/string helpers + file:// URL normalization for OS-opened targets
    chartSpec.ts           Chart JSON parsing/validation/series inference
src-tauri/
  src/lib.rs               All Tauri commands + single-instance + macOS RunEvent::Opened wiring
  src/main.rs              Calls simple_md_lib::run()
  capabilities/default.json  Window permissions (core, dialog, opener, opener:allow-open-path)
  tauri.conf.json          Bundle config + .md/.markdown/.mdown/.mkd file associations
docs/product-and-technical-spec.md   Long-form product+tech spec
```

## Document modes

`DocumentMode = 'preview' | 'source' | 'wysiwyg' | 'split'`. Mapped to UI tabs **Display / Text / Hybrid / Split**.

- `preview` — read-only rendered view (`MarkdownPreview`, react-markdown pipeline).
- `source` — raw Markdown source in CodeMirror (`EditorPane`).
- `wysiwyg` — **labeled "Hybrid"** in the UI. Real editable rendered view powered by Milkdown's Crepe preset (`HybridEditor`). ProseMirror under the hood with commonmark + GFM; lossless Markdown round-trip via `crepe.on(listener => listener.markdownUpdated(...))`.
- `split` — side-by-side: source on left, rendered on right.

Naming gotcha: the **internal value `wysiwyg` displays as "Hybrid"** to the user. The internal name "split" used to be called "hybrid" — it was renamed when the WYSIWYG mode shipped, since "Hybrid" is a clearer label for "edit rich text directly while Markdown stays the source." If you see legacy refs to `'hybrid'` as a mode value, they're stale.

The `HybridEditor` is keyed on `activeDocument.id` so switching documents remounts the editor. Inside the lifecycle the `content` prop is captured **once on mount** — Milkdown owns the document state, we just observe markdown updates via the listener and forward them to `handleContentChange`.

## Tauri commands (Rust → JS)

Defined in `src-tauri/src/lib.rs`, wrapped in `src/lib/desktop.ts`:

- `list_workspace(path)` → `WorkspaceSnapshot { path, name, tree }`. Filters tree to markdown files only.
- `open_document(path)` / `save_document(path, content)` → `DocumentPayload`.
- `create_note(parentDir, name, initialContent?)` — auto-appends `.md` if missing; rejects existing files.
- `create_folder(parentDir, name)`.
- `rename_path(path, newName)` — single leaf only; rejects path separators / `.` / `..`.
- `delete_path(path)` — recursive for directories.
- `fetch_remote_markdown(url)` — HTTPS only, 10 MB cap, 30s timeout, max 5 redirects, normalizes `github.com/.../blob/...` → `raw.githubusercontent.com/...`. Validates content via content-type or path or heuristic clue match.
- `opened_targets()` / `clear_opened_targets()` — drains the queue of paths the OS asked us to open. Combined with the `'opened'` event so files double-clicked in Finder land in the running app.

Markdown file extensions recognized everywhere: `md`, `markdown`, `mdown`, `mkd`.

## Charts: ```chart fenced blocks

`MarkdownPreview` looks for `language-chart` code fences and routes the body to `InlineChart`. The body must be JSON matching `ChartSpec` (`bar` | `line` | `area` | `pie`). Series and xKey are inferred when omitted (`lib/chartSpec.ts`). Errors render an inline `chart-error` panel rather than throwing.

## State conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robrighter/simple-md](https://github.com/robrighter/simple-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
