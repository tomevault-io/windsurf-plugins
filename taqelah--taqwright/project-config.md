---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is the **taqwright library itself** — an E2E mobile testing layer that puts Playwright's runner on top of Appium 3 / WebDriver. Appium 3.x is the supported runtime; 2.x is allowed on a best-effort basis because every `mobile:` command shape is identical between the two majors (`doctor` warns rather than errors). If you add code that uses an Appium 3-only command, branch on `classifyAppiumVersion()` from [src/doctor.ts](src/doctor.ts) and surface a clear message — don't let users on 2.x hit cryptic server errors. Published privately to GitHub Packages (`npm.pkg.github.com`) as `@taqwright/taqwright`; ships a CLI (`bin: taqwright → dist/bin/index.js` — the command name stays `taqwright` even though the package is scoped). The README is the canonical user-facing docs; this file is for working _on_ the library.

There is no `tests/`, `pages/`, or `fixtures/` directory in this repo — those belong to projects _consuming_ taqwright (the library's own unit tests live in `test/`, singular — see Common commands). `tsconfig.json` deliberately excludes those consumer paths so a misplaced project in a monorepo isn't pulled into the lib build.

## Common commands

```bash
npm install
npm run build      # tsc --build → dist/  (also serves as the typecheck)
npm run clean      # tsc --build --clean
```

There is **no separate `typecheck` script** — `npm run build` is the typecheck. Linting (ESLint, flat config in [eslint.config.js](eslint.config.js)) and formatting (Prettier, [.prettierrc.json](.prettierrc.json)) are their own scripts:

```bash
npm run lint            # eslint .
npm run lint:fix        # eslint . --fix
npm run format          # prettier --write .  (rewrites files)
npm run format:check    # prettier --check .  (CI-safe, no writes)
```

In-repo unit tests live in `test/`, run with Node's built-in runner:

```bash
npm test                # npm run build && node --test test/*.test.js
npm run test:coverage   # same, with --experimental-test-coverage
npm run test:watch
```

**Before pushing, run these locally and make them green first** — the same three gates run in CI ([.github/workflows/ci.yml](.github/workflows/ci.yml)), so a push that skips them just fails remotely:

```bash
npm run format:check    # or `npm run format` to auto-fix, then re-check
npm run lint
npm test                # builds (typecheck) + runs the unit suite
```

[src/inspector/ui.ts](src/inspector/ui.ts) is exempt from Prettier ([.prettierignore](.prettierignore)) — it's one giant template literal; format it by hand per the inspector-UI rules below.

Tests use `node:test` + `node:assert/strict` and import the compiled **`dist/`** output, so the build must run first (`npm test` does it; `dist/` is gitignored). Pure-logic modules (config, capabilities, doctor, recorder, locator-suggester, providers) are tested directly; device-driving code is tested by injecting a hand-rolled fake WebDriver `driver` — see `makeFakeDriver` / `makeLocator` / `makeMobile` in [test/fake-driver.js](test/fake-driver.js). Code needing a real device/network/HTTP layer (most `Mobile`/`Locator` methods, the inspector HTTP handler, `runDoctorChecks`, cloud providers, CLI spawn) is intentionally not unit-covered. Unrelated: `npx taqwright test` is the runner for _consumer_ projects with their own `taqwright.config.ts` — it fails in this repo (no config).

ESM-only (`"type": "module"`, NodeNext). All relative imports inside `src/` must include the `.js` extension even though the sources are `.ts` — NodeNext requires it.

## Architecture

### The config-embedding trick ([src/config.ts](src/config.ts))

`defineConfig(TaqwrightConfig)` does **not** return a TaqwrightConfig — it returns a Playwright `TestConfig` with the taqwright shape stashed under a private symbol key (`TAQWRIGHT_KEY = '__taqwright__'`). This lets users pass `taqwright.config.ts` directly to Playwright's `--config` flag.

Critically, the per-project Playwright `use` block intentionally **does not** carry the taqwright `use` options — only `{ taqwrightProject: name }`. The fixture re-reads the user's config from disk at runtime (`loadTaqwrightConfig()`) using that project name as a key. Don't try to forward taqwright use-options through Playwright's `use` field; serializing `RegExp` device names and other rich types across the worker boundary won't work.

### CLI shape ([src/bin/index.ts](src/bin/index.ts))

The CLI splits into delegators and direct commands:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taqelah/taqwright](https://github.com/taqelah/taqwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
