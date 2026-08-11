---
trigger: always_on
description: Desktop GUI for markdown-vdb. Electron + Svelte 5 + CodeMirror 6. Bridges to the Rust CLI (`mdvdb`) via IPC subprocess execution.
---

# Markdown VDB — Electron App

Desktop GUI for markdown-vdb. Electron + Svelte 5 + CodeMirror 6. Bridges to the Rust CLI (`mdvdb`) via IPC subprocess execution.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Electron Main Process                │
│   src/main/index.ts — window lifecycle, IPC handlers    │
│   cli.ts — spawns `mdvdb` binary via execFile           │
│   store.ts — electron-store for collections             │
│   errors.ts — typed error classes for IPC transport     │
├─────────────────────────────────────────────────────────┤
│                     Preload Script                      │
│   src/preload/index.ts — contextBridge API surface      │
│   api.d.ts — TypeScript types for window.api            │
├─────────────────────────────────────────────────────────┤
│                   Renderer (Svelte SPA)                 │
│   components/ — Sidebar, Header, Editor, FileTree, etc. │
│   stores/ — collections, editor, files (Svelte stores)  │
│   lib/ — editor theme, soft render, frontmatter deco    │
│   types/cli.ts — TS interfaces mirroring Rust structs   │
│   styles/ — tokens.css, global.css, app.css             │
└─────────────────────────────────────────────────────────┘
```

## Project Structure

```
├── src/
│   ├── main/                       # Electron main process
│   │   ├── index.ts                # BrowserWindow creation, app lifecycle
│   │   ├── ipc-handlers.ts         # 30+ IPC channel handlers (cli:*, collections:*, fs:*)
│   │   ├── cli.ts                  # CLI binary detection (findCli) & execution (execCommand)
│   │   ├── store.ts                # electron-store schema (collections, window bounds)
│   │   ├── collections.ts          # Path validation, folder picker, init prompt
│   │   └── errors.ts               # CliNotFoundError, CliExecutionError, CliParseError, CliTimeoutError
│   ├── preload/
│   │   ├── index.ts                # contextBridge.exposeInMainWorld('api', ...)
│   │   └── api.d.ts                # MdvdbApi interface + Collection, SearchOptions, IngestOptions
│   └── renderer/
│       ├── main.ts                 # Svelte app mount
│       ├── App.svelte              # Root layout: Sidebar | Header + Editor + StatusBar
│       ├── index.html              # HTML shell with CSP headers
│       ├── components/
│       │   ├── Sidebar.svelte      # Collection list + file tree panel (256px)
│       │   ├── Header.svelte       # Breadcrumb path + dirty indicator + actions
│       │   ├── Editor.svelte       # CodeMirror 6 markdown editor
│       │   ├── FileTree.svelte     # Tree display with expand/collapse, stats
│       │   ├── FileTreeNode.svelte # Recursive tree node (dir or file)
│       │   ├── StatusBar.svelte    # Language, word count, reading time, CLI status
│       │   └── ui/                 # Design system primitives
│       │       ├── Button.svelte   # Primary/secondary, sm/md/lg
│       │       ├── Badge.svelte    # Status badges with semantic colors
│       │       ├── Input.svelte    # Text input with validation
│       │       ├── IconButton.svelte
│       │       └── Kbd.svelte      # Keyboard shortcut display
│       ├── stores/
│       │   ├── collections.ts      # collections, activeCollectionId, collectionStatus
│       │   ├── editor.ts           # isDirty, wordCount, readingTime (derived)
│       │   └── files.ts            # fileTree, selectedFilePath, fileContent, expandedPaths
│       ├── lib/
│       │   ├── editor-theme.ts     # CodeMirror dark theme + syntax highlighting
│       │   ├── soft-render.ts      # Soft markdown preview decorations
│       │   └── frontmatter-decoration.ts  # YAML frontmatter styling
│       ├── styles/
│       │   ├── tokens.css          # CSS custom properties (colors, spacing, typography)
│       │   ├── global.css          # Resets, grain texture, scrollbars
│       │   └── app.css             # App layout (flexbox shell)
│       └── types/
│           └── cli.ts              # TS interfaces mirroring Rust Serialize structs
├── tests/
│   ├── unit/                       # Vitest + @testing-library/svelte
│   ├── integration/                # Vitest integration tests
│   └── e2e/                        # Playwright E2E tests
├── out/                            # Build output (main/, preload/, renderer/)
├── electron.vite.config.ts         # Three-target build config
├── vitest.config.ts                # Unit test config (jsdom)
├── vitest.integration.config.mts   # Integration test config
├── playwright.config.ts            # E2E test config
├── tsconfig.json                   # Workspace references root
├── tsconfig.node.json              # Main + preload (ES2020, commonjs)
├── tsconfig.web.json               # Renderer (ESNext, extends @tsconfig/svelte)
├── eslint.config.mjs               # ESLint 9 flat config + svelte plugin
└── .prettierrc                     # Semi, single quotes, trailing comma es5
```

## Tech Stack

| Layer        | Technology                           | Purpose                                                          |
| ------------ | ------------------------------------ | ---------------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geckse/tesseract-md-app](https://github.com/geckse/tesseract-md-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
