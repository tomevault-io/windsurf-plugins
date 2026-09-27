---
trigger: always_on
description: Guidance for AI agents (and humans) contributing to this repository.
---

# AGENTS.md

Guidance for AI agents (and humans) contributing to this repository.

## Project

**Polire** is a desktop typing assistant that helps users write better text — primarily targeted at people writing in a non-native language.

The app is designed to feel ambient: it runs in the system tray, stays out of the way, and is summoned with a global hotkey (`Ctrl+Alt+P`). From the root palette, `Esc` or the hotkey hides it back into the tray rather than quitting; on secondary views, `Esc` navigates back.

The product name is **Polire**.

## Stack

- **Runtime:** [Electron](https://www.electronjs.org/) 42 (main + renderer processes)
- **UI:** [React](https://react.dev/) 19 + [TypeScript](https://www.typescriptlang.org/) (strict mode)
- **Styling:** [Tailwind CSS](https://tailwindcss.com/) v4 (CSS-first config via `@import "tailwindcss"`, no `tailwind.config.js`)
- **Icons:** [Phosphor Icons](https://phosphoricons.com/) — `@phosphor-icons/react`. Use only this library; do not introduce other icon sets (lucide, heroicons, etc). Always import with the `Icon` suffix (`GearIcon`, `SparkleIcon`, …) — the unsuffixed exports are deprecated and will trigger warnings.
- **Bundler / dev server:** [Vite](https://vitejs.dev/) 6 with [`vite-plugin-electron`](https://github.com/electron-vite/vite-plugin-electron) (simple preset)
- **Package manager / runner:** [Bun](https://bun.sh/) — use `bun install`, `bun run <script>`
- **Formatter / linter:** [Biome](https://biomejs.dev/) — run `bun run format` before committing
- **Target platforms:** Windows and Linux (X11 recommended for global shortcuts; Wayland support is limited)

## Build outputs

- `dist/` — bundled renderer (HTML + JS + CSS)
- `dist-electron/` — bundled main + preload (`main.js`, `preload.mjs`)
- `node_modules/.cache/tsc/` — TypeScript build info (project references)

All three are gitignored.

## Project structure

```
assets/
├── brand/        # vector source mark for Polire branding
├── app/          # native app icon master + Linux size variants
└── tray/         # compact status-area icons (1x and 2x)
electron/
├── ipc/                     # shared IPC sender authorization + handler registration
├── modules/
│   ├── ai/                  # AI prompts, credentials, execution, validation + bridge API
│   ├── app/                 # application-level renderer requests (external links)
│   ├── notes/               # Markdown note persistence, validation + bridge API
│   ├── settings/            # persisted application settings + bridge API
│   ├── tray/                # system tray lifecycle, validation + bridge API
│   ├── update/              # update status, notifications, checks + bridge API
│   └── window/              # window lifecycle, validation + bridge API
├── constants.ts             # shared paths, dimensions, hotkey and URLs
├── preload-api.ts           # composed renderer-facing bridge contract types
├── preload.ts               # bridge entry: exposes module APIs on `window.api`
└── main.ts                  # entry: startup wiring + global shortcuts
src/
├── app.tsx                  # shell: wraps NavProvider + AnimatePresence + view renderer
├── main.tsx                 # React entry — mounts <App /> into #root
├── main.css                 # Tailwind v4 import + dark variant + base styles
├── theme.ts                 # theme state: persistence + DOM apply
├── i18n/                    # local UI translations + locale persistence (en, pt-BR, es)
├── types.ts                 # shared types (CommandOption, View, ...)
├── global.d.ts              # ambient types (window.api from preload)
├── hooks/                   # context consumers + reusable interaction hooks
├── providers/
│   ├── i18n.tsx             # locale context + `t()` translation resolver
│   ├── nav.tsx              # navigation stack (push/pop) + slide direction
│   ├── correction.tsx       # correction request/result state
│   ├── translation.tsx      # translation request/result state
│   └── notes.tsx            # local notes list/create/update/remove state
├── views/
│   ├── palette.tsx          # root view: search + command list (no back)
│   ├── settings.tsx         # settings overview: theme + nav to sub-pages
│   ├── ai-settings.tsx      # AI sub-page: provider list + API key form
│   ├── correction.tsx       # AI correction before/after screen
│   ├── translation.tsx      # AI translation before/after screen
│   └── notes.tsx            # notes controller: loading, editing + shortcuts
└── components/
    ├── ai-settings/         # provider rows/config + API key field
    ├── notes/               # note list, editor, relative dates + footer hints
    ├── palette/             # palette input, commands, list + save feedback
    ├── settings/            # theme option list
    └── ui/                  # shared layout, rows, hints, footer + result screens
index.html         # renderer HTML shell
vite.config.ts     # Vite + plugins (React, Tailwind, Electron)
```

`assets/` contains native runtime resources. Any future packaging configuration must include this directory so window and tray icons remain available outside development.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gugeldev/polire](https://github.com/gugeldev/polire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
