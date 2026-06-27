---
trigger: always_on
description: Guidance for AI assistants (Claude Code, etc.) working in this repository.
---

# CLAUDE.md

Guidance for AI assistants (Claude Code, etc.) working in this repository.

## Project

**boardown** is a small open-source task board that stores its data as markdown
files inside the project repo. It is aimed at solo developers and follows a
lightweight scrum flow: a Backlog plus releases with a `future → current →
finished` lifecycle, and epics as a cross-release grouping that doubles as
the storage container for unscheduled tasks. The product spec lives in
[PRODUCT.md](./PRODUCT.md) — read it before making non-trivial changes.

License: MIT.

## Communication rules

- Write all code, comments, identifiers, commit messages, and documentation in
  **English**.
- When replying to the user in chat, **reply in the same language the user
  wrote in**. Do not translate the user's message just to process it — answer
  in their language directly.

## Tech stack (decided)

- **Language:** TypeScript (strict mode everywhere)
- **Frontend framework:** React 18
- **Build tool:** Vite
- **Package manager / monorepo:** pnpm with workspaces
- **State management:** Zustand (kept minimal — single-user app, no Redux)
- **Schema validation:** Zod (frontmatter + config)
- **Markdown frontmatter:** `gray-matter`
- **Drag & drop:** `@dnd-kit/core`
- **Tests:** Vitest

The primary MVP distribution channel is a **VS Code extension** (implemented
and packaged into an installable `.vsix`; see PRODUCT.md roadmap), which reads
`.boardown/` from the open workspace. The
**browser shell (`packages/web`) is a development and local-from-sources tool**
— it boots `@boardown/ui` against a selected `.boardown/` over a Vite
middleware, and is not a production distribution channel for the MVP. File
System Access API integration and a folder picker are explicit non-goals for
the MVP.

## Repo layout

```
boardown/
├── packages/
│   ├── core/          # platform-agnostic logic: schemas, md parser, FsAdapter
│   │                  # interface, board operations, ID generator
│   ├── ui/            # React app: components, Zustand store, UI flow.
│   │                  # Takes an FsAdapter as a prop. No DOM-only / Node /
│   │                  # VS Code imports.
│   ├── web/           # Dev-only browser shell: Vite app, DevHttpFsAdapter
│   │                  # over a Vite middleware that serves a selected
│   │                  # .boardown/, manual Reload only. Mounts @boardown/ui.
│   │                  # No production browser deployment in MVP.
│   ├── vscode/        # Primary MVP shell: extension host (esbuild) + webview
│   │                  # (Vite) hosting @boardown/ui. Built in stages; see
│   │                  # PRODUCT.md roadmap.
│   ├── electron/      # Desktop shell (macOS / Windows / Linux): Electron main
│   │                  # (esbuild) + renderer (Vite) hosting @boardown/ui over a
│   │                  # Node FsAdapter. Post-MVP.
│   └── cli/           # Command-line / agent-facing shell: maps argv onto
│                      # @boardown/core board-ops over a Node FsAdapter, with
│                      # machine-readable JSON output. Post-MVP.
├── package.json
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── CLAUDE.md
└── PRODUCT.md
```

`@boardown/core` and `@boardown/ui` are both consumed source-only:
`main`/`exports` point at `src/index.ts`, no separate build step. The shell's
bundler (Vite for `web`, esbuild for the VS Code host) transpiles them, and
`tsc`/ESLint resolve their types straight from source. Neither package emits a
`dist/`. Only the shells (`web`, `vscode`, `electron`, `cli`) have a `build`
script — they bundle the source-only libraries into their own artifacts.

`packages/vscode` is the primary MVP distribution target, built bottom-up in
stages (see PRODUCT.md roadmap). It is a sibling shell next to `web` and reuses
`@boardown/ui` unchanged — only the `FsAdapter` implementation and entry flow
differ. The extension host is bundled with esbuild (`vscode` external, CJS) and
the webview with Vite; both run in the Extension Development Host via F5. The
webview mounts the real `@boardown/ui` with a `VsCodeFsAdapter` that proxies
`read/write/list/stat` to the host over `postMessage`, where the host serves
them from `vscode.workspace.fs`. The board root is the single open workspace
folder's `.boardown/`; choosing among multiple roots or an arbitrary folder is
out of scope (Electron territory). An Electron build is post-MVP and follows the
same shell pattern.

`packages/web` ships a small Vite middleware that exposes
`/api/fs/{read,list,stat,write}` over HTTP, scoped to a selected `.boardown/`
folder, plus a `DevHttpFsAdapter` that talks to those
endpoints. This is the **only** browser-side path for the MVP — it is the
working environment for `@boardown/ui` development, not a stepping stone to
a production browser app. A production browser shell (with the FS Access
API or otherwise) is post-MVP and may or may not happen.

`packages/cli` is a post-MVP shell that does **not** mount `@boardown/ui` — it has
no DOM. Instead it consumes `@boardown/core` directly (board-ops, loader,
serializer, schemas) and implements `FsAdapter` over `node:fs/promises`, mapping
CLI commands onto board operations. It is aimed at agents and scripts: output is a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grinev/boardown](https://github.com/grinev/boardown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
