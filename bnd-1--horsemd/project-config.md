---
trigger: always_on
description: HorseMD is an Electron + Vite + React Markdown editor with a shared renderer for desktop and Capacitor mobile builds.
---

# Repository Guidelines

## Project Structure & Module Organization

HorseMD is an Electron + Vite + React Markdown editor with a shared renderer for desktop and Capacitor mobile builds.

- `src/main/index.js`: Electron main-process lifecycle, window, IPC assembly, menus, assets, and update checks.
- `src/main/filesystem.js`, `watchers.js`, `documents.js`: file operations, watchers, dialogs, and PDF export.
- `src/preload/index.js`: secure `window.api` bridge exposed to the renderer.
- `src/renderer/src/`: React app, Milkdown editor, hooks, shell components, themes, i18n, platform shim.
- `src/renderer/src/components/Editor.jsx`: Crepe/ProseMirror editor wrapper; keep new editor features in focused `editor-*.js` helpers when possible.
- `src/renderer/src/hooks/` and `src/renderer/src/lib/`: file ops, lifecycle, outline, find/replace, menus, and review actions.
- `docs/`: architecture, features, development workflow, mobile notes, and manual testing.
- `scripts/`: lightweight verification and CDP helpers.
- `website/`: static product/download homepage; `guide/`: isolated VitePress user tutorial site.
- `build/`, `icons/`, `android/`, `ios/`: packaging assets and mobile shells.

## Build, Test, and Development Commands

```bash
npm install
npm run dev
npm run build
npm start
npm run dist
npm run build:mobile
npm run test:source-map
npm run guide:check
node scripts/test-strike-guard.mjs
```

- `npm run dev`: starts Electron/Vite hot reload.
- `npm run build`: builds main, preload, and renderer into `out/`; CI uses this.
- `npm start`: runs the built app.
- `npm run dist`: creates the host-platform installer via `electron-builder`.
- `npm run build:mobile`: builds the Capacitor renderer into `dist-mobile/`.
- `npm run test:source-map`: runs Markdown raw-offset ↔ ProseMirror mapping tests for tables, duplicate text, code, images, lists, and HTML.
- `npm run guide:check`: validates tutorial metadata, versions, links, assets, screenshot privacy/dimensions, and builds the guide site.
- `node scripts/test-strike-guard.mjs`: runs CriticMarkup strike regression checks.

## Coding Style & Naming Conventions

Use ES modules, React functional components, two-space indentation, single quotes, and no semicolons. Components use `PascalCase.jsx`; hooks use `useName.js`; editor helpers use descriptive `editor-*.js` names. No formatter or linter is enforced, so match nearby code. Avoid adding large logic blocks to `App.jsx` or `Editor.jsx`; prefer small modules under `hooks/`, `lib/`, or `components/editor-*.js`.

## Testing Guidelines

There is no single `npm test` command. Run `npm run build` before PRs. For editor/review logic, add or update focused scripts under `scripts/`. For UI changes, follow `docs/manual-test-checklist.md`; CDP helpers are in `docs/development.md`.

CDP automation must use `scripts/lib/electron-test-app.mjs` with its default
background mode so tests do not take native keyboard focus or show over the
user's work. Input-rule, caret, line-break, mode-switch, and source-fidelity
tests must send committed text one character at a time via
`scripts/lib/human-input.mjs`, with raw key events for delimiters and special
keys. Bulk text insertion is only appropriate for paste semantics, fixture
setup, or behavior unrelated to incremental typing. Per-character committed
Chinese text is not a substitute for a real IME composition test.

For any real/manual reproduction of rich-text ↔ source divergence, launch the
actual HorseMD process with `--horsemd-input-trace` in Electron argv before the
user starts editing. Do not rely on `ELECTRON_ENABLE_LOGGING` alone: it does not
persist the source/candidate/canonical transaction evidence needed to identify
the first divergence. Keep the trace-enabled instance running through the real
operation sequence, and inspect the first integrity failure before changing code.

User-facing changes must update the matching `guide/` page. Tutorial screenshots must come from a rebuilt and freshly installed current app using an isolated profile; follow `docs/user-guide-maintenance.md`. Never publish screenshots containing personal paths or stale UI.

## Commit & Pull Request Guidelines

History uses concise subjects such as `feat(#38): ...`, `fix(site): ...`, `docs: ...`, `chore: ...`, and `refactor: ...`. Keep commits focused and imperative. PRs should describe the change, link issues, include UI screenshots, mention desktop/mobile impact, and add `CHANGELOG.md` entries for user-facing changes.

## Security & Configuration Tips

Keep Electron renderer access behind `window.api`; do not enable Node integration. Do not commit signing keys, keystores, or local config such as `android/key.properties`. When adding native capabilities, update both desktop preload and the Capacitor shim or gate the UI with `window.api.capabilities`.

## User Execution Preference

- For routine project-development commands such as build, start, dev, test, read-only diagnostics, and local packaging, the user has explicitly authorized automatic execution without asking again each time.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BND-1/horseMD](https://github.com/BND-1/horseMD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
