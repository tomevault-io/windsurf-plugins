---
trigger: always_on
description: This file is the repository-level development contract. Every agent working in this
---

# Cocode Agent Engineering Rules

This file is the repository-level development contract. Every agent working in this
repository MUST read and follow it before changing code. User instructions for a
specific task take precedence when they explicitly conflict with this file; otherwise
these rules are mandatory.

## 1. Agent instruction portability

When adding or revising agent constraints, prompts or examples in this file:

- MUST NOT include personal names, usernames, email addresses, machine names,
  user-home paths or other person/device-specific identifiers.
- MUST use repository-relative paths, environment variables, standard tool
  discovery or neutral placeholders so the instructions can be reused directly
  on another device.
- MUST NOT assume a particular operating system, shell, checkout location or
  locally installed absolute binary path. If a platform-specific step is
  unavoidable, state the condition and provide a discoverable alternative.
- MUST keep the resulting instructions independent of local environment state;
  do not encode secrets, local credentials, or assumptions about another
  developer's filesystem.

## 2. Project baseline

This repository is an electron-vite + electron-builder + TypeScript desktop application.

- Node.js: `>=22.12.0` (use the version in `.nvmrc` when available).
- pnpm: `10.34.5` exactly.
- Electron: `43.x` as pinned by `package.json`.
- Renderer: React `18.x`.
- Styling: Tailwind CSS `3.x` with PostCSS.
- UI primitives: shadcn/ui source components backed by Radix UI.
- Class composition: `clsx` + `tailwind-merge` through `cn()`.

Before running project commands, use the repository runtime. If `.nvmrc` exists,
select the version declared there; otherwise use a compatible Node.js version
from the `engines` field. Run pnpm through Corepack at the pinned version:

```bash
corepack pnpm@10.34.5 <command>
```

Do not weaken `engines` constraints or upgrade React to 19 / Tailwind to 4 as a
shortcut for a local environment mismatch.

## 3. Source tree and ownership

The source tree is organized by Electron runtime boundary first, then by business
boundary:

```text
src/
├── main/                  # trusted Electron main process
├── preload/               # minimal, allow-listed context bridge
├── renderer/              # React renderer process
├── contracts/             # cross-process protocol and DTO definitions
└── shared/                # pure TypeScript code safe for every runtime
```

### `src/main`

Main owns privileged desktop capabilities and the authoritative business model.

```text
src/main/
├── index.ts               # thin process entry; calls bootstrap only
├── bootstrap/             # composition root and dependency wiring
├── shell/                 # Electron lifecycle and desktop shell adapters
├── contexts/              # main-process bounded contexts
└── shared/                # main-only technical capabilities
```

`main/shell` may contain Electron APIs, BrowserWindow management, menus, tray,
protocols, updater, shortcuts, security and lifecycle code. It MUST NOT contain
business rules that belong in a bounded context.

`main/index.ts` MUST remain thin. Move lifecycle, window creation and registrations
to focused modules under `bootstrap` or `shell`.

### `src/preload`

Preload is the only bridge between privileged Main APIs and Renderer code.

```text
src/preload/
├── index.ts               # thin preload entry
├── bridges/               # allow-listed APIs grouped by capability/context
├── validators/            # runtime validation of IPC inputs/outputs
└── types/                 # Window/global declarations for exposed APIs
```

Preload MUST expose narrow capability APIs through `contextBridge`. Never expose
`ipcRenderer`, `ipcMain`, Node.js modules, or a generic `send/invoke` wrapper.
Preload MUST NOT implement domain rules or become a second application service layer.

### `src/renderer`

Renderer owns React presentation, user interaction and renderer-local application
state. It MUST NOT import Electron or Node.js privileged APIs.

```text
src/renderer/
├── index.tsx              # React 18 createRoot entry
├── app/                   # renderer composition root, providers, router, layouts
├── contexts/              # UI-side bounded contexts
├── components/ui/         # shadcn/ui source components
├── hooks/                 # renderer-wide hooks used by multiple contexts
├── lib/                   # renderer-wide technical helpers, including cn()
├── shared/                # renderer-only generic UI/state utilities
└── styles/                # Tailwind layers, tokens and global styles
```

`src/renderer/app/App.tsx` is the application shell. It may compose providers and
routes, but business workflows belong in a context.

### `src/contracts`

`contracts` defines how independent runtime boundaries communicate:

```text
src/contracts/
├── ipc/                   # channel names, request/response DTOs
├── events/                # cross-boundary event payloads
└── schemas/               # runtime schemas for boundary validation
```

Contracts describe what is sent, received and returned. They MUST NOT contain
repositories, database adapters, Electron calls, business implementations or UI
components.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cocode-agency/cocode](https://github.com/cocode-agency/cocode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
