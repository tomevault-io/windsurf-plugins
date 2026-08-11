---
trigger: always_on
description: OpenScreen is a free, open-source screen recorder and video editor (Electron + React + TypeScript + Pixi.js) maintained as a continuation of the original v1.5.0 release. This file is the canonical guide for any AI coding agent working in this repo.
---

# AGENTS.md

OpenScreen is a free, open-source screen recorder and video editor (Electron + React + TypeScript + Pixi.js) maintained as a continuation of the original v1.5.0 release. This file is the canonical guide for any AI coding agent working in this repo.

## Setup commands

- Install deps: `npm install` (Node 22.22.1, npm 10.9.4 — see `package.json#engines`)
- Start dev:    `npm run dev` (Vite dev server; Electron window opens via `vite-plugin-electron`)
- Build:        `npm run build` (TypeScript check + Vite build + electron-builder)
- Typecheck:    `npx tsc --noEmit` — app code only. CI also runs `npx tsc -p tsconfig.test.json --noEmit` in a separate job ("Typecheck (tests)"), so **run both**: test files are invisible to the root config, and a type error in a `*.test.ts` fails CI while the root check stays green.
- Test (unit):  `npx vitest --run <path>` while you work, `npm run test` once at the end — see [Testing instructions](#testing-instructions)
- Test (e2e):   `npm run test:e2e` (Playwright)
- Lint:         `npm run lint` (Biome 2.4)
- Format:       `npm run format` (Biome, tabs, double quotes, 100-col)
- i18n check:   `npm run i18n:check` (validates the 13 locale files)

## Project layout

- `src/` — React app: UI, editor components, timeline, i18n, captioning/cursor/exporter libs
- `electron/` — main process, IPC, recording orchestration
- `electron/native/` — **native** capture helpers: `screencapturekit/` (Swift, macOS) and `wgc-capture/` (C++/Win32, Windows). These are built and shipped with the app, not loaded from npm
- `technical-documentation/` — architecture, engineering and testing reference (start at its README)
- `tests/` — Playwright e2e specs + fixtures
- `scripts/` — native build scripts, diagnostic tools
- `nix/`, `flake.nix` — Linux packaging
- `release/`, `dist-electron/` — build artifacts (gitignored)

## Code style

- TypeScript strict mode (`tsconfig.json`). No `any` (Biome `noExplicitAny` is `warn` — don't add new `any`).
- Biome handles lint AND format. Tabs, double quotes, 100-col width, LF line endings. Run `npm run lint:fix` before committing.
- React functional components only. Hooks at top level (Biome `useHookAtTopLevel` is `error`).
- Imports: use the `useImportType` discipline (Biome organizes them).
- Husky + lint-staged runs Biome on staged `*.{ts,tsx,js,jsx,mts,cts,json}`.
- The repo is pre-1.x and not production-grade — rough edges are expected, but new code should be clean.

## Testing instructions

### When to run what

The full unit suite is ~1670 tests over 140 files and takes over a minute. Running it after
every edit is the main way an agent turns a 5-minute task into a 30-minute one, so don't:

- **While you work** — run only what you touched: `npx vitest --run src/lib/foo.test.ts`,
  or `npx vitest --run src/lib/ai-edition` for a directory. `npm run test:changed` picks
  the affected files off the working tree, `npx vitest --run --changed main` off the
  branch diff. A single file is 1–10s against ~80s for everything.
- **Typecheck and lint freely** — `npx tsc --noEmit` and `npm run lint` are seconds, not
  minutes. They are the right inner-loop check, not the test suite.
- **Once, at the end** — `npm run test` before you commit or open the PR. One full run per
  task, not per edit. If the change is narrow and CI will run anyway, the targeted run plus
  CI is enough; say so rather than burning the wall-clock twice.
- **Never** `npm run test:watch` — it does not terminate, and it will hang the session.

### Layout and conventions

- Unit tests live next to source as `*.test.ts` / `*.test.tsx` (Vitest). Config is
  `vitest.config.ts`; it covers `src/`, `electron/` and `.github/`.
- **The default environment is `node`.** A test that needs a DOM opts in with
  `// @vitest-environment jsdom` on line 1 — that is also the fix for `document is not
  defined`. Don't add it to a test that doesn't need it: jsdom setup dominates this
  suite's runtime (see the comment in `vitest.config.ts`).
- Anything platform-conditional (`process.platform`) must pin the platform in the test.
  CI is Linux-only, so a Linux-only code path left unpinned is green in CI and red on
  every Windows and macOS machine — `electron/recording/webm-seek-index.test.ts` is the
  worked example.
- E2E tests are in `tests/e2e/` (Playwright). Some specs are platform-specific (e.g. `windows-native-checklist.spec.ts`).
- Add a test for every new behavior in the same package as the code under test.
- All tests must pass before opening a PR. CI runs `npm run test` on every PR.

## Desktop E2E testing with computer-use

Unit/browser tests can't exercise real capture (native screen recording, a physical webcam, the tray). To verify a recording/editor feature end to end, drive the actual Electron app with the **computer-use** MCP (screenshot + click/type on the desktop). This is the required "manual smoke test on real Windows/macOS" for native changes.

**Launch the app**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getopenscreen/openscreen](https://github.com/getopenscreen/openscreen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
