---
trigger: always_on
description: Peacock is a Visual Studio Code extension that subtly changes the color of your workspace. It's ideal when you have multiple VS Code instances, use VS Live Share, or use VS Code's Remote features and want to quickly identify which editor is which.
---

# Peacock — Agent Guide

## Project Overview

Peacock is a Visual Studio Code extension that subtly changes the color of your workspace. It's ideal when you have multiple VS Code instances, use VS Live Share, or use VS Code's Remote features and want to quickly identify which editor is which.

- **Publisher:** johnpapa
- **VS Code Marketplace:** [Peacock](https://marketplace.visualstudio.com/items?itemName=johnpapa.vscode-peacock)
- **Version:** See `package.json` for the current version

## Repository Structure

```
vscode-peacock/
├── src/                        — Extension source code (TypeScript)
│   ├── extension.ts            — Extension entry point (activation, command registration)
│   ├── commands.ts             — Command implementations (enter color, random, favorites, etc.)
│   ├── apply-color.ts          — Core color application logic
│   ├── color-library.ts        — Color manipulation utilities (uses tinycolor2)
│   ├── configuration/          — VS Code configuration read/write helpers
│   ├── models/                 — TypeScript interfaces, enums, constants, state
│   ├── inputs.ts               — User input prompts (color picker, quick pick)
│   ├── statusbar.ts            — Status bar color display
│   ├── live-share/             — VS Live Share integration
│   ├── remote/                 — Remote development integration
│   ├── test/                   — Mocha unit tests
│   ├── logging.ts              — Output channel logging
│   ├── mementos.ts             — Global state persistence
│   ├── notification.ts         — User notifications
│   └── object-library.ts       — Object/element management
├── e2e/                        — Playwright end-to-end tests (docs screenshots)
├── docs/                       — Docsify documentation site
│   ├── guide/                  — User guide pages
│   ├── about/                  — About pages
│   ├── changelog/              — Changelog page
│   ├── _sidebar.md             — Navigation sidebar
│   └── index.html              — Docsify entry point
├── resources/                  — Extension icons and assets
├── testworkspace/              — Test workspace used by Mocha tests
├── .github/
│   ├── workflows/docs.yml      — Docs deploy + Playwright e2e tests
│   ├── ISSUE_TEMPLATE/         — Bug report and feature request templates
│   └── PULL_REQUEST_TEMPLATE/  — PR template
├── webpack.config.js           — Webpack bundler config (Node + Web targets)
├── tsconfig.json               — TypeScript configuration
├── playwright.config.ts        — Playwright e2e test config
└── package.json                — Extension manifest, commands, settings, scripts
```

## Tech Stack

- **Language:** TypeScript (strict mode)
- **Runtime:** VS Code Extension Host (Node.js + Browser via `extension-web.js`)
- **Bundler:** Webpack (dual target: `extension-node.js` and `extension-web.js`)
- **Color library:** tinycolor2 — all color manipulation goes through this
- **VS Live Share:** `vsls` package for Live Share integration
- **Linting:** ESLint + Prettier (husky pre-commit hook runs Prettier automatically)
- **Testing:** Mocha (unit tests in `src/test/`) + Playwright (e2e docs tests in `e2e/`)
- **Docs:** Docsify (static site in `docs/`, deployed to GitHub Pages)

## Build & Run

```bash
npm install                     # Install dependencies
npm run webpack                 # Build (development mode)
npm run vscode:prepublish       # Build (production mode — webpack --mode production)
```

To run the extension locally, press **F5** in VS Code — this launches the Extension Development Host with the extension loaded.

## Testing

```bash
npm test                        # Compile + run Mocha unit tests
npm run just-test               # Run Mocha tests only (skip compile)
npm run test:e2e                # Run Playwright e2e tests (docs screenshots)
npm run test:coverage           # Run tests with Istanbul coverage
npm run test-all                # Run unit + Live Share tests
```

**Test structure:**
- Unit tests live in `src/test/` and use Mocha + Sinon for mocking
- E2e tests live in `e2e/` and use Playwright to capture docs screenshots
- The `testworkspace/` directory is used as a VS Code workspace during tests

**Test requirements:**
- Every bug fix must include a regression test that fails without the fix and passes with it
- Every new feature must include unit tests covering the happy path and relevant edge cases
- Never merge code that reduces the passing test count

## Key Patterns and Conventions

- **Commands** are registered in `extension.ts` and implemented in `commands.ts`
- **Color application** flows through `apply-color.ts` → `color-library.ts` (tinycolor2)
- **Configuration** is read/written via helpers in `src/configuration/`
- **State** is managed through VS Code's `workspaceState` and `globalState` APIs (see `mementos.ts`)
- **Models** define all TypeScript interfaces and enums in `src/models/`
- **Dual output** — the extension compiles to both Node (`extension-node.js`) and Web (`extension-web.js`) via Webpack

## CI/CD


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnpapa/vscode-peacock](https://github.com/johnpapa/vscode-peacock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
