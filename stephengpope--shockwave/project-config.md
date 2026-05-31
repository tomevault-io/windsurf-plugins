---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — start electron-vite (`electron-vite dev --watch --remoteDebuggingPort=9222`). Builds main + preload, serves the renderer on :5173, launches Electron, and auto-reloads on any `src/**` change. CDP for the renderer is exposed on :9222.
- `npm run build` — production build to `out/` (main, preload, renderer).
- `npm start` — `electron-vite preview` against the production build.
- `npm run dist` — build then `electron-builder` (produces dmg/nsis/AppImage per `build` block in `package.json`).
- `npm test` — run the test suite (node:test, no install needed).
- `npx eslint .` — lint (config in `eslint.config.js`). No npm script; run via `npx`.

## TypeScript

The codebase is TS with a deliberately permissive posture: `noImplicitAny: false` in `tsconfig.json`, no strict mode. Conversions favor "kill `.jsx` extensions" over fully-typed annotations — many components use `any` for state. Treat that as the migration baseline, not a target.

Vite has an `extensionAlias` (in `electron.vite.config.js`) so imports written as `./foo.js` resolve to `./foo.ts` at dev/build time. That's why the source still has `.js` / `.jsx` import strings — don't "fix" them.

Cross-process types live in `src/shared/`:
- `api.d.ts` — the typed `window.api.*` surface (what preload exposes).
- `settings.ts` — typed `Settings` shape and `DEFAULT_SETTINGS`.
- `constants.ts` — shared enums + `APP_NAME`.

For day-to-day workflow (when to restart, how to read main vs renderer logs, how to attach to the renderer via CDP for headless debugging, IPC discipline), use the **electron-dev** skill at `.claude/skills/electron-dev/SKILL.md`.

## Architecture

Electron app with a Vite + React 19 renderer. The renderer is a markdown-workspace editor (CodeMirror 6) with wiki-links (`[[name]]`), backlinks, tabs, drafts, multiple workspaces, a force-graph view, a live-preview / raw view-mode toggle, an editor status bar, bookmarks, daily notes, quick search, image embeds, voice transcription, and a right-hand coding-agent chat sidebar (pi).

### Process boundary

- **Main** (`src/main/`): filesystem, dialogs, context menus, settings persistence + secret encryption, `nativeTheme`, the file watcher + rename correlator, the `app://media/...` protocol for serving workspace images, window-bounds persistence, the pi coding-agent session, skill-library management, the agent-tokens pi extension, the AssemblyAI voice-token mint. All IPC handlers are registered here. Entry: `src/main/main.ts`. **Deep doc: `src/main/CLAUDE.md`.**
- **Preload** (`src/preload/preload.cjs`): exposes a single `window.api` surface (typed in `src/shared/api.d.ts`). The renderer never touches Node — every fs/dialog/agent call goes through `window.api.*`. Also exposes `webUtils.getPathForFile` so the renderer can resolve drag-dropped folder paths (skill import).
- **Renderer** (`src/renderer/`): React app rooted at `main.tsx` → `App.tsx`. App.tsx is thin orchestration; heavy state lives in hooks under `src/renderer/hooks/`. Vite root is `src/renderer/` (configured in `electron.vite.config.js`); build output goes to `out/renderer/`. Built main/preload land at `out/main/index.js` and `out/preload/index.cjs`. **Deep doc: `src/renderer/CLAUDE.md`.**

### Where things live

| Area | File(s) | Deep doc |
|---|---|---|
| Main-process internals (watcher, IPC, settings, app://, coding agent, voice token, GitHub sync engine) | `src/main/*.ts` | `src/main/CLAUDE.md` |
| Renderer internals (hooks, editor decorations, chat sidebar, voice, bookmarks, daily notes, quick search, sync UI) | `src/renderer/**` | `src/renderer/CLAUDE.md` |
| GitHub sync | `src/main/sync.ts`, `src/main/syncEngine.ts`, `src/renderer/settings/SyncSection.tsx`, `src/renderer/settings/WorkspaceSyncDialog.tsx` | "GitHub sync" sections in both subdocs |
| Cross-process types + constants | `src/shared/{api.d.ts, settings.ts, constants.ts}` | this file, below |
| Tests | `tests/*.test.js` | `tests/CLAUDE.md` |

## Terminology

The canonical names. Use these in UI strings, comments, docs, agent prompts — anywhere a human (user or contributor) might read them.

- **File** — a `.md` document in the workspace. The user-facing noun for the thing you create / open / edit / delete. **Never use "page" or "note"** — both were earlier conventions that have been retired.
- **Basename** — a file's name with no folder path and no `.md` extension. For `notes/projects/Foo.md`, the basename is `Foo`. This is what wiki-links use, and what the link index is keyed by.
- **Workspace** — the folder on disk the user has opened. Everything inside it (files, images, other assets) is part of the workspace. Code sometimes still says "vault" (Obsidian-inherited); new code uses "workspace".
- **Wiki-link** — the `[[Some File]]` syntax linking one file to another by basename. The term comes from MediaWiki/Obsidian/etc. Variants: `[[File#Heading]]`, `[[File|Display]]`. Resolution is workspace-wide, case-insensitive, basename-only — never include a folder path. The parser + index live in `src/renderer/linkIndex.js` (mirrored in main at `src/main/linkParser.js`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stephengpope/shockwave](https://github.com/stephengpope/shockwave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
