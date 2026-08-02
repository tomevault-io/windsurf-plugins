---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An unofficial Electron desktop client that wraps Notion's web apps (Notion, Calendar, Mail) to give Linux users a native-like experience: custom titlebar, multi-tab browsing, system tray, native notifications, and self-updating packages. Linux-only — there is no Windows/macOS path.

## Commands

- `npm start` — run the app locally in dev mode (`APPIMAGE=/ electron-vite dev`, with HMR for the renderer). The `APPIMAGE=/` env is what the updater uses to detect AppImage mode.
- `npm run build` — bundle main/preload/renderer with **electron-vite** into `out/`.
- `npm run preview` — run the built app from `out/` (production-like; no dev server).
- `npm run typecheck` — `tsc --noEmit` across the three project tsconfigs (node, preload, web). No JS is emitted by `tsc`; electron-vite/esbuild does the transpiling.
- `npm run lint` — ESLint `--fix` + Prettier on **changed/untracked files only** (not the whole tree).
- `npm run lint:all` — same but across all git-tracked files.
- `npm run lint:deep` — lint files changed since the merge-base with the main branch.
- `npm run make` — `electron-vite build` then package an unpacked Linux x64 build via `electron-packager` into `dist/`.
- `npm run pack` — `electron-vite build` then build distributable rpm/deb/AppImage via `electron-builder`.
- `npm run dev:server` — start the local release server (`dev/release-server.mjs`) used to test the auto-updater against fake releases.
- Flatpak/Snap: `npm run flatpak-build` / `flatpak-test` / `flatpak-lint` and `snap-build` / `snap-upload` (shell scripts under `dev/`).

There is **no test suite** — `npm test` intentionally exits 1. Don't add references to running tests.

Requires Node 22+ / npm 10+.

## TypeScript & build (important)

The app is **TypeScript bundled with electron-vite**. There is no longer a "run the `.mjs` directly" path.

- Source lives under `src/`: `src/main/` (main process), `src/preload/` (preload scripts), `src/renderer/` (renderer pages + their entry scripts), and `src/shared/` (types shared by preload + renderer). The package is ESM (`"type": "module"`).
- `electron.vite.config.ts` defines three build targets. Output goes to `out/{main,preload,renderer}`; `package.json` `main` points at `out/main/index.js`. **Preload scripts are emitted as `.cjs`** (`output.format: 'cjs'`) because Electron's sandboxed renderers only accept CommonJS preloads.
- Type checking is separate from bundling: `tsconfig.node.json` (main), `tsconfig.preload.json` (preload, includes the DOM lib for `docs-preload`), `tsconfig.web.json` (renderer), with the root `tsconfig.json` referencing all three. All use **aggressive strict** settings (`strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noImplicitOverride`). Run `npm run typecheck` after changes.
- Runtime static assets (icons, etc.) stay in `assets/` and are resolved via helpers in `src/main/lib/resources.ts` (`resolveAsset`, `resolvePreload`, `loadRendererPage`) — not via `__dirname` paths. `assets/` ships through electron-builder `extraResources`; the renderer copies it via Vite `publicDir`.

## Linting workflow (important)

Linting is driven by custom Node scripts in `dev/scripts/`, **not** by calling `eslint`/`prettier` directly:
- `lint.mjs` collects the relevant file list from git (via `helpers/git.mjs`), filters to `ALLOWED_EXTENSIONS` (`dev/scripts/helpers/constants.mjs`), then runs `eslint -c ./eslint.config.ts --fix` and re-stages.
- The ESLint config is **flat config written in TypeScript** (`eslint.config.ts`), so it must be invoked with `-c ./eslint.config.ts`.
- Husky installs a `pre-commit` hook (committed at `.husky/pre-commit`) running `npm run typecheck` then `npm run lint`. CI (`.github/workflows/ci.yml`) runs typecheck + lint:all + build on push/PR.

## Code style

- **Self-explanatory code over comments.** Write code that explains itself and avoid comments. Only comment when it's necessary — to explain a hack, an unorthodox approach, or a magic number, or to document the public API of an internal library. Keep necessary comments short.
- **Use the `private` / `public` keywords** for class members. Do not mark private methods or properties with the `#` symbol.
- **Don't extract a class method that's only called once** — inline it. The single exception is when inlining would push the caller's **cognitive complexity over 10** (the `complexity` ESLint threshold, `max: 10`); then a `private` helper is warranted. Relatedly, a public method that never uses `this` is a stateless helper and belongs in `src/main/lib/`, not on a class (enforced by `publicMethods/public-class-methods-use-this`).

## Architecture

All application logic runs in the **Electron main process** (ESM). There is no renderer framework; renderer UI is plain HTML in `src/renderer/` (each page has a sibling `.ts` entry) bundled by electron-vite and driven by preload scripts.

### Wiring (`src/main/index.ts`)
`src/main/index.ts` is the composition root. It:
1. Acquires a single-instance lock (second launch focuses the existing window).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anechunaev/notion-electron](https://github.com/anechunaev/notion-electron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
