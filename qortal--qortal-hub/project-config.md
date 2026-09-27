---
trigger: always_on
description: Desktop/web/mobile client for the Qortal blockchain, plus Reticulum-based
---

# Qortal Hub — agent guide

Desktop/web/mobile client for the Qortal blockchain, plus Reticulum-based
peer-to-peer chat and voice. GPL-3.0. React 19 + TypeScript + Vite 7 + MUI 7,
wrapped by Capacitor into an Electron desktop app and an Android app.

The repo is a single npm project at the root (the renderer) with a **second,
separate npm project in `electron/`** (the Electron main process). They have
their own `package.json` and `node_modules`. Changes to renderer code only reach
the desktop app after a `cap sync`.

## Commands

Root (renderer):

| Command                 | What                                                                     |
| ----------------------- | ------------------------------------------------------------------------ |
| `npm run dev`           | Vite dev server on `http://localhost:5173/` — browser only, no Reticulum |
| `npm run build`         | Production build into `dist/`                                            |
| `npm test`              | Vitest watch mode                                                        |
| `npx vitest run <path>` | Run one test file — preferred while iterating                            |
| `npm run coverage`      | Vitest with coverage                                                     |
| `npm run lint`          | ESLint, `--max-warnings 0`                                               |
| `npm run format`        | Prettier over the whole repo                                             |

Desktop (needs Python 3.9+ and Git on PATH):

```bash
npm run build                                # or npm run dev for the web part
npx cap sync @capacitor-community/electron   # REQUIRED after web/native changes
cd electron && npm install && npm run electron:start
```

Packaging lives in `electron/`: `electron:make-win`, `electron:make-mac`,
`electron:make-lin-docker`, `electron:make-arm-docker`. Linux release artifacts
should use the Docker variants so native binaries build against an older glibc.

There is no typecheck script; `tsc` runs via `vite-plugin-checker` during dev.

## Layout

```text
src/
  components/   382 files — UI, grouped by feature (Chat/, Group/, Apps/, QortalLand/, ...)
  lib/          call/, dm/, group-call/, reticulum/, webrtc/ — transport + protocol logic
  hooks/        useReticulumGroupChat, useVoiceCall, useAuth, ...
  atoms/        Jotai global state
  contexts/     React contexts (GroupCallContext, CallSwitchGuardContext, ...)
  i18n/         i18next setup + locales/<lang>/<namespace>.json
  qortal/       Qortal Core API calls, q-app request handling
  qdn/          QDN publish/fetch + encryption
  transactions/ signed blockchain transactions
  background/   POW + background tasks
  utils/        events bus, storage, chat helpers
  test/         Vitest setup + mocks for Electron/native modules
electron/src/   Electron main: reticulum-*.ts, group-call.ts, chat-db.ts, core.ts
electron/resources/presence_bridge.py   Python bridge to the Reticulum daemon
docs/           architecture docs — read these before touching Reticulum or calls
```

### The Reticulum path

```text
Renderer (React)  → window.electronAPI.reticulum*() / window.groupCall.*()
Electron main     → reticulum-daemon.ts, reticulum-bridge.ts, group-call.ts
Python bridge     → presence_bridge.py  (fd3/fd4 binary IPC)
rnsd              → Reticulum Network Stack daemon
```

Reticulum features **do not work in the browser dev server** — they need the
Electron shell. See `docs/reticulum.md` and `docs/group-audio-calls.md`.

## Conventions

**i18n is mandatory.** Every user-visible string goes through i18next — including
`aria-label`, placeholders, tooltips, and error messages. See the `i18n` skill in
`.agents/skills/i18n/` and `docs/i18n_languages.md`. Large parts of
`components/QortalLand/` and the `Reticulum*` chat components are still
hardcoded English and are being migrated; do not copy that pattern.

**State.** Jotai atoms in `src/atoms/` for global state. Cross-component
signalling uses a DOM CustomEvent bus, not a store:

```ts
import {
  executeEvent,
  subscribeToEvent,
  unsubscribeFromEvent,
} from '../utils/events';
```

Always unsubscribe in the effect cleanup.

**Styling.** MUI `sx` with `theme.palette.*` — light and dark themes both ship
(`src/styles/theme-light.ts`, `theme-dark.ts`). Do not hardcode hex colors.

**Formatting.** Prettier: single quotes, semicolons, 80 columns, 2 spaces, es5
trailing commas. Run `npm run format` rather than hand-aligning.

**TypeScript is not strict** — `strict: false`, `noImplicitAny: false`. Don't
assume null-safety is enforced by the compiler; check at runtime.

**Tests.** Vitest + jsdom, ~144 test files colocated next to their subject as
`*.test.ts(x)`. Files under `electron/**` run in the `node` environment instead.
Native and Electron modules are aliased to fakes in `src/test/mocks/` — extend
those rather than mocking ad hoc. Test setup: `src/test/setup.ts`.

## Agent config — skills and commands

**Definitions live in `.agents/`; `.claude/` symlinks to them.** Claude Code
reads only `.claude/`, so the links are what make the config load.

```text
AGENTS.md                          the file you are reading
CLAUDE.md            → AGENTS.md   loaded unconditionally every session


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Qortal/Qortal-Hub](https://github.com/Qortal/Qortal-Hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
