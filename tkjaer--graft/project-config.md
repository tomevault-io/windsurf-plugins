---
trigger: always_on
description: A collaborative markdown editor backed by GitHub. It provides a Google Docs-like editing experience for markdown files stored in GitHub repos — WYSIWYG editing (Tiptap), inline comments anchored to text ranges, and change suggestions. The underlying format is always raw markdown committed to a Git branch.
---

# Graft — Collaborative Markdown Editor

## What is this?

A collaborative markdown editor backed by GitHub. It provides a Google Docs-like editing experience for markdown files stored in GitHub repos — WYSIWYG editing (Tiptap), inline comments anchored to text ranges, and change suggestions. The underlying format is always raw markdown committed to a Git branch.

Available as both a **VS Code extension** and a **standalone web app** (hosted on GitHub Pages).

## Why does this exist?

Markdown files in GitHub repos (docs, RFCs, specs, blog posts) are collaborative documents, but the editing experience is poor:

- GitHub's web editor is basic and has no inline commenting on text ranges
- PR review comments are line-based and tied to diffs — they can't anchor to phrases
- Non-technical collaborators (PMs, writers, designers) shouldn't need to know markdown syntax
- There's no "suggest changes" flow for prose like Google Docs has

Graft bridges this gap: developers keep their markdown-in-git workflow, non-technical collaborators get a rich editor with inline comments and suggestions.

## Architecture

Two entry points share the same core logic:

```
┌──────────────────────┐    ┌─────────────────────────┐
│  VS Code Extension   │    │  Web App (GitHub Pages)  │
│  (EditorPanel.ts)    │    │  (web/main.ts)           │
│                      │    │                          │
│  ┌────────────────┐  │    │  ┌───────────────────┐   │
│  │  Tiptap WYSIWYG│  │    │  │  Tiptap WYSIWYG   │   │
│  │  (WebviewPanel)│  │    │  │  (same editor)    │   │
│  └───────┬────────┘  │    │  └───────┬───────────┘   │
│          │ postMsg   │    │          │ direct calls   │
│  ┌───────▼────────┐  │    │  ┌───────▼───────────┐   │
│  │  github/api.ts │  │    │  │  web/api.ts        │   │
│  │  (Node/Buffer) │  │    │  │  (browser/btoa)    │   │
│  └───────┬────────┘  │    │  └───────┬───────────┘   │
└──────────┼───────────┘    └──────────┼───────────────┘
           │                           │
     ┌─────▼───────────────────────────▼──┐
     │         GitHub REST API            │
     │  Content: branch file              │
     │  Comments: orphan branch           │
     └────────────────────────────────────┘

Shared modules: src/shared/anchoring.ts, src/shared/url.ts, src/types.ts
```

### Key design decisions

1. **Tiptap + tiptap-markdown** — WYSIWYG editing with markdown serialization. Users never see markdown syntax unless they open the source pane. The file on disk is always `.md`.

2. **Split view with CodeMirror 6** — A toggleable source pane shows the raw markdown side-by-side with the WYSIWYG editor. Built with CodeMirror 6 (`@codemirror/view`, `@codemirror/lang-markdown`). Changes sync bidirectionally with debouncing. Optional vim bindings via `@replit/codemirror-vim` (toggled at runtime using a CodeMirror `Compartment`). Scroll position syncs proportionally between panes (with a 50ms guard + `Math.round()` to prevent feedback loops).

2. **Comments on an orphan branch** — Comments are stored as JSON files on a `graft-comments` orphan branch in the same repo. This avoids polluting the main branch, requires zero infrastructure (no database), and comments get full Git history for free. The branch is created automatically on first comment.

3. **Text anchoring, not line numbers** — Comments are anchored by `{ text, prefix, suffix }` — the highlighted text plus ~50 chars of surrounding context. This survives edits (lines added/removed, paragraphs reworded). Resolution algorithm: exact match → fuzzy prefix/suffix match → orphaned. Core logic is in `src/shared/anchoring.ts`.

4. **Comments die with the branch** — When a PR branch is deleted after merge, the comments path (keyed by branch name) becomes unreachable. No cleanup needed.

5. **Two entry points, shared core** — The VS Code extension uses `postMessage` to bridge the webview and Node APIs. The web app calls the GitHub API directly from the browser. Both share types, anchoring logic, and URL parsing from `src/shared/`.

6. **Device flow auth (web app)** — The web app uses GitHub's OAuth device flow, which requires no client secret and runs entirely in the browser. The GitHub App client ID is the only config, set via `VITE_GITHUB_CLIENT_ID` at build time. Tokens are stored in `sessionStorage` (cleared on tab close).

7. **No backend** — Both the extension and web app are fully client-side. The web app is a static site on GitHub Pages. GitHub handles auth, storage, and access control.

8. **Themes** — The web app supports multiple themes (GitHub Light, GitHub Dark, GitHub Dark Dimmed, Dracula, Solarized Light) defined in `src/shared/themes.ts`. Each theme is a set of CSS variable values applied to `:root` via JS. Defaults to system preference, persisted in `localStorage`. The VS Code extension uses VS Code's own theme via `--vscode-*` CSS variables.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tkjaer/graft](https://github.com/tkjaer/graft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
