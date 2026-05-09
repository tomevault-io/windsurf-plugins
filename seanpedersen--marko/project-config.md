---
trigger: always_on
description: A Tauri-based markdown editor with Obsidian-style WYSIWYG inline editor.
---

# Marko

A Tauri-based markdown editor with Obsidian-style WYSIWYG inline editor.

## Tech Stack

- **Frontend**: Svelte 5 (with runes), TypeScript, Vite
- **Backend**: Tauri 2 (Rust)
- **Editor**: CodeMirror 6 with custom live preview extension
- **Styling**: Tailwind CSS v4 (`@tailwindcss/vite`) + CSS variables for theming

## Project Structure

```
src/
├── app.html                          # HTML shell
├── styles.css                        # Global styles
├── routes/
│   ├── +layout.ts                    # SvelteKit layout (SSR disabled)
│   └── +page.svelte                  # Root page (mounts MarkdownViewer)
├── assets/
│   ├── favicon.svg
│   └── icon.png
├── lib/
│   ├── MarkdownViewer.svelte         # Main app container (layout, file I/O, keybindings, auto-save)
│   ├── Installer.svelte              # First-run installer flow
│   ├── Uninstaller.svelte            # Uninstall flow
│   ├── components/
│   │   ├── codemirror/
│   │   │   ├── livePreview.ts        # Obsidian-style live preview (hides syntax when cursor away)
│   │   │   ├── wikiLinkCompletion.ts # Wiki-link autocomplete extension for [[...]] syntax
│   │   │   └── theme.ts             # CodeMirror theme using CSS variables
│   │   ├── CodeMirrorEditor.svelte   # Editor wrapper (CodeMirror 6 instance)
│   │   ├── EditorHeader.svelte       # Obsidian-style header with back/forward nav and collapsed breadcrumb path
│   │   ├── TableOfContents.svelte    # TOC sidebar (220px, parses headings)
│   │   ├── FolderExplorer.svelte     # File tree sidebar (220px, recursive dir listing)
│   │   ├── TitleBar.svelte           # Window title bar (traffic lights, sidebar toggles, tabs, theme, window controls)
│   │   ├── TabList.svelte            # Horizontal tab strip
│   │   ├── Tab.svelte                # Single tab component
│   │   ├── HomePage.svelte           # Start screen (recent files/folders)
│   │   ├── Modal.svelte              # Confirmation dialog (save/discard/cancel)
│   │   ├── SettingsModal.svelte      # Settings dialog (editor width, sidebar position, CLI install)
│   │   ├── KanbanBoard.svelte        # Kanban board UI (Obsidian + Marko extended format)
│   │   ├── CardDetailPane.svelte     # Card detail modal (editable title + CodeMirror body editor)
│   │   └── ContextMenu.svelte        # Right-click context menu
│   ├── stores/
│   │   ├── tabs.svelte.ts            # TabManager class: tab CRUD, navigation history, dirty state
│   │   └── settings.svelte.ts        # SettingsStore class: editor prefs persisted to localStorage
│   └── utils/
│       ├── debounce.ts               # Typed debounce with call()/cancel()
│       ├── parseHeadings.ts          # Extract headings from markdown (with line numbers)
│       ├── frontmatter.ts            # YAML frontmatter parser
│       ├── kanban.ts                 # Shared kanban types, single-pass parser, Obsidian serializer
│       ├── markoKanban.ts            # Marko-extended serializer (card bodies, marko-kanban-plugin key)
│       └── wikiLinks.ts              # Wiki-link utilities (file index, resolution, fuzzy matching)
```

## Key Components

### MarkdownViewer (`src/lib/MarkdownViewer.svelte`)
- Main app shell: manages layout, file loading/saving, keyboard shortcuts, drag-and-drop
- Auto-save: debounced (300ms) via `debounce` utility, controlled by `settings.autoSave`
- **File watching**: watches the active file via `watch_file` (notify crate); on external change, re-reads from disk and updates editor (debounced 300ms). Save guard (`ignoringFileChange` flag, 500ms cooldown) prevents feedback loops. Tabs with unsaved edits skip reload. On tab switch, stale content is detected by comparing disk vs in-memory content.
- Sidebar layout: TOC and FolderExplorer overlay the editor; editor reflows only when viewport is narrow (uses `clamp()` on `left` to account for 720px content max-width + 2rem padding)
- TOC button visibility depends on `hasHeadings` derived (only shown when document has headings)
- Wiki-links: builds file index from folder contents, handles `marko:wiki-link` click events, resolves links and creates missing files

### CodeMirror Editor (`src/lib/components/CodeMirrorEditor.svelte`)
- Props: `value`, `readonly`, `theme`, `onchange`, `zoomLevel`, `fileType`, `editorWidth`, `fileIndex`
- Exports: `scrollToLine(lineNumber)`, `findHeadingLine(text, level, occurrence)`
- Uses `EditorView.lineWrapping` for automatic line wrapping
- Content layout: `.cm-scroller` has `padding: 2rem`, `.cm-content` has `max-width` set via `--editor-max-width` CSS variable
- Wiki-link autocomplete via `wikiLinkCompletion()` extension

### Live Preview (`src/lib/components/codemirror/livePreview.ts`)
- Hides markdown syntax (**, ##, [](), etc.) when cursor is NOT on that line
- Reveals syntax when cursor enters the line
- Applies visual styling via CodeMirror decorations
- Supports wiki-links: `[[filename]]` or `[[filename|display text]]` syntax
- **Important**: Avoid using CSS margins on line decorations — they break click position calculations. Use `line-height` for spacing instead.

### Wiki-Links (`src/lib/utils/wikiLinks.ts` + `wikiLinkCompletion.ts`)
- Obsidian-style internal linking between markdown files
- Syntax: `[[target]]` or `[[target|display text]]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeanPedersen/Marko](https://github.com/SeanPedersen/Marko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
