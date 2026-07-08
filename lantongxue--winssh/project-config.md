---
trigger: always_on
description: **Updated:** 2026-05-26 · **Commit:** 50c93b3 · **Branch:** main
---

# AGENTS.md

**Updated:** 2026-05-26 · **Commit:** 50c93b3 · **Branch:** main

Cross-platform SSH/SFTP desktop client. Electron 39 + React 19 + TypeScript 5 + Vite 7 + Tailwind 4. Core runtime: ssh2, node-pty, better-sqlite3, xterm.js, Monaco, electron-updater.

> **Child AGENTS.md**: `src/main/`, `src/renderer/src/`, `test/`, `web/` — read domain-specific docs there.

## STRUCTURE

```
src/main/           # Electron main process (Node.js) — see child AGENTS.md
src/preload/        # contextBridge typed IPC — 2 files only (index.ts + index.d.ts)
src/renderer/src/   # React workbench app — see child AGENTS.md
src/shared/         # Domain models (NOT just types): 12 files incl. Zod schemas, API surface
themes/builtin/     # VSCode-style JSON theme packs (loaded by main, not CSS in renderer)
test/               # Mirror-tree test infra — see child AGENTS.md
web/                # Fully independent brand-site subproject — see child AGENTS.md
scripts/            # ESM (.mjs) build/utility scripts (mock update feed)
build/              # NSIS installer.nsh + platform icons + macOS entitlements
```

**Non-obvious**: `src/renderer/src/` double nesting is electron-vite convention. No `pages/` dir — entire app is a single workbench shell. `src/native/` referenced in README but removed — do not use.

## WHERE TO LOOK

| Task                     | Location                                                                                          | Notes                                            |
| ------------------------ | ------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| Add new IPC method       | shared/types → shared/validation → main/ipc/register-_ → preload → features/_/api → query-keys    | 6-layer cross-cutting (see below)                |
| Modify session lifecycle | main/session-manager + renderer/store/sessions-store + workbench-context                          | SessionManager is 2897 lines — most complex file |
| Change theme             | shared/themes → main/theme-registry → themes/builtin/_/themes/_.json → renderer/lib/theme         | Theme packs are JSON, not CSS                    |
| Add server table column  | main/database + main/application/servers-app-service + shared/validation                          | Also update ServerUpsertInput/serverSchema       |
| Add SFTP feature         | main/session-manager editor file streaming API (`openFileReadStream`/`openFileWriteStream`/`writeFileChunk`/`closeFileWriteStream`/`cancelFileStream`) + renderer/features/sftp/api + workbench-sftp-\*-editor | SFTP editor streaming is text-only              |
| Work on UI layout        | renderer/components/workbench/                                                                    | Keep-mounted for session/local-terminal editors  |
| Write renderer tests     | test/renderer/ + test/renderer/helpers/create-winssh-api                                          | Never co-locate tests in src/                    |
| Work on web/ site        | web/src/                                                                                          | Separate package.json, tsconfig, vite config     |

## Commands

```bash
npm run dev              # Electron dev mode (hot reload)
npm run build            # typecheck + electron-vite build (NOT just vite build)
npm run test             # All Vitest tests for desktop app
npm run typecheck        # Both Node + Web tsc projects
npm run lint             # KNOWN to have historical failures - do not treat as blocking
npm run format           # Prettier

npx vitest run test/path/to/file.test.ts      # Single test file
npx vitest run -t "pattern name"              # Filter by test name

npm run web:dev          # web/ subproject dev server (separate Vite build)
npm run web:test         # web/ subproject tests (separate vitest)
npm run updates:mock     # Generate mock update feed for local testing
npm run updates:serve    # Serve mock update feed on localhost
npm run dist:win         # Build Windows NSIS installer
```

**Verification order**: `npm run typecheck` → `npm run test` (lint is not a gate)

## Path Aliases

```
@/          → src/renderer/src/
@main/      → src/main/
@renderer/  → src/renderer/src/
@shared/    → src/shared/
@test/      → test/
```

Use these aliases in imports; both src and tests consume them.

## Architecture

**Process boundary**: `src/main/` (Node.js/Electron main) ↔ `src/preload/` (bridge) ↔ `src/renderer/src/` (React). Shared types live in `src/shared/`.

- **`src/main/index.ts`** is a thin entry (`app.whenReady().then(bootstrap)`). All assembly is in **`src/main/bootstrap.ts`**.
- **`src/main/application/`** is a use-case orchestration layer (servers/sessions/settings services).
- **`src/main/ipc/`** has 4 registrars: `register-server-ipc.ts`, `register-session-ipc.ts`, `register-system-ipc.ts`, `register-command-history-ipc.ts`.
- **`src/renderer/src/features/*/api`** is the ONLY approved entry point for renderer code to call preload bridge. ESLint enforces this: `window.winsshApi` is banned in `src/renderer/src/**` except in `features/shared/api/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lantongxue/winssh](https://github.com/lantongxue/winssh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
