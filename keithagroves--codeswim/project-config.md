---
trigger: always_on
description: A standalone Electron desktop app for navigating a codebase as a hierarchy
---

# Diagram-First Code Navigator (Electron app)

A standalone Electron desktop app for navigating a codebase as a hierarchy
of mermaid diagrams. The original of the idea — `codeswim-vscode`
is the same idea ported to a VS Code extension, and `codeswim-example`
is the demo content both target.

The thesis (from [plan.md](./plan.md)): as AI generates more code, humans
should navigate architecture through intentional diagrams, not by reading
generated implementation. Diagrams live as markdown files; this app makes
them clickable.

## Monorepo layout

This is an npm-workspaces + Turborepo monorepo. The Electron app lives in
`apps/desktop`; framework-free logic is extracted into `packages/*` and
consumed as TypeScript source — there is no per-package build step,
electron-vite bundles the workspace source directly:

- `@codeswim/contract` — IPC payload types + the `DiagramNavApi` surface;
  the versioned seam between main, preload, and renderer.
- `@codeswim/domain-git` / `-github` / `-kanban` / `-skills` — Electron-free
  domain logic. Electron-coupled pieces (auth secret storage, `shell.open`,
  the built-in skills dir) are *injected* by `apps/desktop/src/main` rather
  than imported, so the packages stay portable.
- `@codeswim/coverage` — the diagram/source drift checker.
- `@codeswim/commit` — prompt-commit synthesis + triage.
- `@codeswim/harness` — the opencode plugin, session gate, tools, and
  prompts that get bundled into `out/harness/`.

Run everything from the repo root: `npm run dev` / `build` / `typecheck` /
`test` delegate to the desktop app or fan out across packages via Turbo.

## Commands

|                                         |                                                                         |
| --------------------------------------- | ----------------------------------------------------------------------- |
| `npm run dev`                           | electron-vite dev server with HMR (renderer) and rebuild (main/preload) |
| `npm run build`                         | typecheck + production build into `out/`                                |
| `npm run typecheck`                     | runs `typecheck:node` (main + preload) then `typecheck:web` (renderer)  |
| `npm run lint`                          | eslint, cached                                                          |
| `npm run format`                        | prettier --write                                                        |
| `npm run build:mac` / `:win` / `:linux` | electron-builder packaging                                              |

## Process layout

Standard Electron three-process app:

- **Main** ([apps/desktop/src/main/index.ts](apps/desktop/src/main/index.ts)) — owns the filesystem.
  Opens the folder picker, reads files, runs the chokidar watcher, spawns
  npm scripts via `child_process.spawn`, walks the directory tree. All IPC
  handlers (`pick-folder`, `read-file`, `list-markdown`, `list-tree`,
  `watch`/`unwatch`, `read-package-scripts`, `run-script`/`kill-script`)
  live here.

- **Preload** ([apps/desktop/src/preload/index.ts](apps/desktop/src/preload/index.ts) +
  [index.d.ts](apps/desktop/src/preload/index.d.ts)) — minimal `contextBridge`
  surface. Keeps `contextIsolation: true`, `nodeIntegration: false`.

- **Renderer** ([apps/desktop/src/renderer/src/App.tsx](apps/desktop/src/renderer/src/App.tsx)) — React app.
  Owns all UI state via reducer + context. Components consume `useStore()`.

The IPC contract lives in the `@codeswim/contract` package
([packages/contract/src/api.ts](packages/contract/src/api.ts)).
Treat it as a versioned interface — adding a method requires touching all
three processes.

## Renderer architecture

State is a single reducer in [state.tsx](apps/desktop/src/renderer/src/state.tsx),
exposed via context defined in [store.ts](apps/desktop/src/renderer/src/store.ts). The
two files are split so vite's fast-refresh works (only-components rule).

Views render diagrams or Markdown explanations; source code is never shown
inside Codeswim. The current implementation file is tracked as a relative
posix path (`currentFile`), while `currentDocumentPath` is the Markdown
document being rendered. Breadcrumbs are a stack; navigation pushes onto it,
"back" / clicking a crumb pops to that point.

Source links continue to target real files for coverage. The main process
resolves them to `.codeswim/explanations/<source-path>.md` (with adjacent
Markdown fallbacks). The header's "Open in editor" command opens the actual
source file through Electron.

Path resolution lives in [path-utils.ts](apps/desktop/src/renderer/src/path-utils.ts).
Renderer code never deals in absolute paths — it converts to absolute only
at the IPC boundary, so diagrams stay portable.

## Mermaid integration

[DiagramView.tsx](apps/desktop/src/renderer/src/components/DiagramView.tsx) initializes
mermaid with `securityLevel: 'loose'` (required for `click ... call
navigate(...)` to invoke `window.navigate`). Mermaid is rendered
imperatively via `mermaid.render()`; `startOnLoad` is off.

The webview-style CSP in `apps/desktop/src/renderer/index.html` needs
`'unsafe-eval'` in `script-src` for mermaid loose mode and inline styles
allowed for the SVG output. If you tighten CSP, verify mermaid still

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keithagroves/codeswim](https://github.com/keithagroves/codeswim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
