---
trigger: always_on
description: These instructions tell Copilot how to produce code that fits the Chamber codebase. They reflect patterns that already exist here. When in doubt, prefer consistency with surrounding code over external best practices.
---

# GitHub Copilot Instructions — Chamber

These instructions tell Copilot how to produce code that fits the Chamber codebase. They reflect patterns that already exist here. When in doubt, prefer consistency with surrounding code over external best practices.

## Priority Guidelines

1. **Version compatibility** — match the exact versions pinned in `package.json` and `.nvmrc`. Never use APIs newer than what is installed.
2. **Architecture first** — respect the dependency direction described in *Architecture* below. A change that violates it is wrong, even if it compiles.
3. **Read these files first** when context is needed: `AGENTS.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, `tsconfig.json`, `eslint.config.mjs`.
4. **Codebase patterns over invention** — scan neighboring files (same folder, same kind of test, same kind of service) and copy their shape. Do not introduce a pattern that does not already exist somewhere in `src/`.
5. **Security boundaries are non-negotiable** — see *Security* below.

## Stack & Versions (pinned)

- **Runtime**: Node `24.15.0` (`.nvmrc`), Electron `41`
- **Language**: TypeScript `6.0`, ESM source (CommonJS module target in `tsconfig.json`), `target: ESNext`, `strict: true`, `noImplicitAny: true`, `moduleResolution: bundler`, JSX `react-jsx`
- **Path alias**: `@/*` → `./src/*` (tsconfig + `config/vitest.config.ts` + `components.json`)
- **UI**: React `19`, Tailwind CSS `4`, shadcn/ui style `radix-nova` (Radix primitives + CVA + `tailwind-merge` + `clsx`), Lucide icons
- **Markdown**: `react-markdown` + `remark-gfm` + `rehype-highlight` + `highlight.js`
- **Build**: Electron Forge `7.11` + `@electron-forge/plugin-vite` + Vite `8`
- **Testing**: Vitest `4`, `@testing-library/react`, `@testing-library/jest-dom`, `jsdom`
- **Lint**: ESLint `10` flat config, `typescript-eslint` `8`, `eslint-plugin-import-x`
- **Copilot SDK**: `@github/copilot-sdk@0.3.0` and `@github/copilot@1.0.40-1` — both **pinned exactly**, not ranged. The runtime is committed under `chamber-copilot-runtime/` and shipped in the package; do not bump either without a coordinated changelog entry.
- **Other**: `keytar@7` (credentials), `croner@10` (cron), `radix-ui@1`, `class-variance-authority@0.7`

> **Invariant**: `tsconfig.json` deliberately leaves `ignoreDeprecations` disabled. Fix deprecation warnings, do not suppress them.

## Architecture

Chamber is an Electron desktop app with three layers under `src/`:

```
src/
  main.ts            # Composition root — wires factories → services → IPC
  preload.ts         # Bridge — exposes a typed API to the renderer
  renderer.tsx       # Renderer entry
  main/              # Electron main process
    services/        # Business logic, organized by capability
    ipc/             # Thin ipcMain.handle adapters (one file per service)
    integration/     # Cross-service integration tests
    contextMenu/, tray/, assets/, wireLifecycleEvents.ts
  renderer/          # React UI
    App.tsx, components/, hooks/, lib/, index.css, env.d.ts
  shared/            # Types and utilities used by both main and renderer
  tests/             # Cross-cutting regression tests and Playwright E2E smoke tests
```

### Dependency direction

- **Renderer → Shared** ✅ (types only, no Electron imports)
- **Main → Shared** ✅
- **Main → Renderer** ❌ (never)
- **Shared → Main or Renderer** ❌ (never)
- **Services** depend on injected ports/factories, not on Electron globals. The composition root (`src/main.ts`) is the *only* place that constructs and wires services. IPC adapters are thin and parameter-injected; they call services, not vice versa.

### Service layout

Each capability lives in its own folder under `src/main/services/<capability>/`:

```
ChatService.ts        # the service
ChatService.test.ts   # colocated unit tests
TurnQueue.ts          # collaborators in the same folder
TurnQueue.test.ts
index.ts              # barrel — public surface only
types.ts              # local types when shared inside the folder
```

Existing capabilities: `a2a`, `auth`, `canvas`, `chamberTools`, `chat`, `chatroom` (with `orchestration/` subfolder for strategies + `approval-gate.ts` + `observability.ts`), `config`, `cron`, `genesis`, `lens`, `mind`, `sdk`.

Add a new capability the same way. Do not flatten services into `src/main/`.

### IPC pattern

IPC adapters are pure plumbing (`src/main/ipc/<name>.ts`):

```ts
// chat.ts — thin adapters for ChatService
import { ipcMain, BrowserWindow } from 'electron';
import type { ChatService } from '../services/chat/ChatService';

export function setupChatIPC(chatService: ChatService, mindManager: MindManager): void {
  ipcMain.handle('chat:send', async (event, mindId: string, message: string, ...) => {
    // marshal args, call the service, forward events via webContents.send
  });
}
```

Conventions:

- Channel names are `lowercase:colon` (`chat:send`, `chat:event`, `mind:list`, `lens:get`).
- Adapters take services as constructor-style parameters; they do **not** new-up services.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ianphil/chamber](https://github.com/ianphil/chamber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
