---
trigger: always_on
description: Operational guide and deep context for any AI coding agent (Claude, etc.) working
---

# CLAUDE.md

Operational guide and deep context for any AI coding agent (Claude, etc.) working
in this repository. Read this first. It explains **what Limboo is**, **how the
code is organized**, **the rules you must follow**, and **what is and is not built
yet**.

> Companion document: [`project.md`](project.md) holds the full product/architecture
> vision. `CLAUDE.md` (this file) is the practical, code-level contract for working
> in the repo. When the two disagree about *current reality*, trust `CLAUDE.md`.

---

## 1. What is Limboo?

Limboo is a **local-first desktop application** that acts as the *operating system
for AI software development*. It is **not an AI model**. Instead, it provides the
environment around a connected coding agent: project management, sessions, file
watching, repository indexing, git operations, terminal execution, memory,
permissions, context, search, and UI.

Core idea: **every development task happens inside a Session**. A session bundles a
repository, branch, chat history, agent, terminal history, checkpoints,
permissions, context, memory, tasks, and generated files into one workspace.

Guiding principles (from `project.md` §4): Fast, Local, Private, Modular, Secure,
Responsive, Observable, Predictable, Recoverable. There is **no backend** — the
only network traffic is the connected coding agent talking to its AI provider.

---

## 2. Tech stack (current)

| Layer            | Choice                                      |
| ---------------- | ------------------------------------------- |
| Shell / desktop  | **Electron 42** (via **Electron Forge 7**)  |
| Bundler          | **Vite 5** (`@electron-forge/plugin-vite`)  |
| UI framework     | **React 19**                                |
| Language         | **TypeScript** (`~4.5`)                      |
| Styling          | **Tailwind CSS v4** (CSS-first, no config)  |
| State            | **Zustand 5** (slice-per-domain stores)     |
| Icons            | **lucide-react**                            |
| Packaging/makers | Squirrel (win), ZIP (mac), deb, rpm (linux) |

Notes / gotchas:

- `@vitejs/plugin-react` is pinned to the **v4** line on purpose. v6 requires
  Vite 8, but Electron Forge's Vite plugin pins **Vite 5** — installing v6 breaks
  peer resolution. If you bump Vite, re-check this.
- Tailwind v4 is **CSS-first**: there is **no `tailwind.config.js`** and **no
  `postcss.config.js`**. All design tokens live in an `@theme` block inside
  [`src/renderer/styles/index.css`](src/renderer/styles/index.css). The Vite
  plugin (`@tailwindcss/vite`) handles PostCSS/autoprefixer internally.
- TypeScript is old (`~4.5`). The renderer is transpiled by **esbuild via Vite**,
  not `tsc`, so type errors do **not** block the dev/build run. **Do not** rely on
  `tsc --noEmit` to verify — TS 4.5 cannot even parse the modern bundled
  `@types/node`. Verify instead with `npx vite build --config
  vite.renderer.config.mts` + `npm run lint` (and esbuild bundles for main/preload).
- **Path aliases**: `@` → `src` and `@shared` → `src/shared`. Configured in all
  three Vite configs (`resolve.alias`) and `tsconfig.json` (`paths`). ESLint's
  `import/no-unresolved` is set to ignore `^@/` and `^@shared/` (the pinned
  ESLint 8 toolchain can't take the TS resolver plugin).
- **Zustand 5** drives renderer state. It's transpiled by esbuild so the old TS
  version is a non-issue.

---

## 3. Project structure

```
limboo/
├── CLAUDE.md                  # you are here
├── project.md                 # full product/architecture vision
├── index.html                 # renderer HTML entry (script → src/renderer/main.tsx)
├── forge.config.ts            # Electron Forge config (entries: src/main/index.ts, src/preload/index.ts; icon)
├── vite.main.config.ts        # Vite config: main process build (+ @ / @shared alias)
├── vite.preload.config.ts     # Vite config: preload build (+ @ / @shared alias)
├── vite.renderer.config.mts   # Vite config: renderer (React + Tailwind + alias) — see note
├── tsconfig.json              # TS config (jsx: react-jsx, DOM libs, path aliases)
├── .eslintrc.json             # ESLint (typescript + import; ignores @ aliases)
├── assets/                    # static assets bundled with the app
│   ├── icon.svg               # source Orbit mark (lucide geometry, transparent, solid accent)
│   ├── icon.png               # 512px window/app icon (rsvg-convert from icon.svg)
│   └── tray.png               # 32px tray icon
├── package.json
└── src/
    ├── global.d.ts            # ambient types for window.limboo (from preload)
    ├── shared/                # code shared across ALL processes
    │   ├── ipc-channels.ts    #   IpcChannels (invoke) + IpcEvents (push) name constants
    │   ├── types.ts           #   AppSettings, WindowStateData, Session, FileChange, CommandId, …
    │   └── constants.ts       #   DEFAULT_SETTINGS, limits, clamp()
    ├── main/                  # MAIN process (Node / OS owner)
    │   ├── index.ts           #   entry: lifecycle, single-instance, CSP, wires managers + IPC
    │   ├── logger.ts          #   file+console logger + global uncaught handlers
    │   ├── storage.ts         #   atomic JSON read/write under userData
    │   ├── paths.ts           #   assetPath() resolver

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BotCoder254/limboo](https://github.com/BotCoder254/limboo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
