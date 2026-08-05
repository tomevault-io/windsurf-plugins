---
trigger: always_on
description: tslog is a TypeScript logger for browsers, Node.js, Deno, and Bun, React Native, and workers. It supports JSON and pretty-printed output, stack traces, error formatting, custom transports, and sub-loggers. v5 adds an always-`pretty` default `type` (colorization and `pretty.passObjectsNatively` — on by default in real browsers — are the env-aware parts, never the format), flat fields-first JSON, async-context correlation, path/regex masking, a middleware chain, and pino/otel/genai presets. Curren
---

## Project Overview

tslog is a TypeScript logger for browsers, Node.js, Deno, and Bun, React Native, and workers. It supports JSON and pretty-printed output, stack traces, error formatting, custom transports, and sub-loggers. v5 adds an always-`pretty` default `type` (colorization and `pretty.passObjectsNatively` — on by default in real browsers — are the env-aware parts, never the format), flat fields-first JSON, async-context correlation, path/regex masking, a middleware chain, and pino/otel/genai presets. Current version: 5.x (the `5` line; `_logMeta.v` in JSON output is `5`).

### Settings are grouped (v5)

There are no flat settings keys. Configuration is organized into groups passed to `new Logger({ ... })`:

- Top-level: `type` (`"json" | "pretty" | "hidden"`, defaults to `"pretty"`), `name`, `minLevel` (name or 0..6), `prefix`, `customLevels`, `strictConfig`, `clock` (injectable `() => Date`).
- `mask: { keys, paths, regex, caseInsensitive, placeholder, censor }` — redact secrets/PII/prompts by key, dotted path (`*` = one segment), or regex.
- `json: { messageKey, levelKey, levelIdKey, timeKey, time, errorKey, numericLevel, stableKeyOrder }` — structured-output key names/shape (`time`: `"iso" | "epoch" | false | fn`).
- `pretty: { enabled, template, errorTemplate, style, timeZone, styles, levelMethod, inspectOptions, ... }`.
- `stack: { capture: "off" | "lazy" | "auto" | "full", internalFramePatterns }`.
- `meta: { property, attachContext }`.

New runtime surface: `getSubLogger`/`child` (aliases), `runInContext`/`getContext` (AsyncLocalStorage correlation), `attachTransport`, `use` (middleware), `flush`, `addLevel`, `isLevelEnabled`, `[Symbol.asyncDispose]`/`[Symbol.dispose]`, `Logger.fromEnv`, and the `defineConfig` helper. Presets ship as subpaths: `tslog/presets/pino`, `tslog/otel`, `tslog/presets/genai`; transports as `tslog/transports/{file,http,ringbuffer}`.


## Quick Reference

```bash
npm install          # Install dependencies
npm test             # Run Vitest test suite
npm run test:browser # Run Playwright browser tests (Chromium, Firefox, WebKit)
npm run test:bun     # Run Vitest under Bun
npm run test:deno    # Build + run Deno test adapter
npm run build        # Full build: types → ESM → browser IIFE bundle → prepare dist
npm run lint         # Biome lint check
npm run format       # Biome format all files
npm run check        # Biome lint + format (write)
npm run coverage     # Run tests with coverage report
npm run test:e2e-apps # Framework E2E: real Next.js (Turbopack) + TanStack Start dev servers (needs network + prior build)
```

## Build System

- **ESM-only.** There is no CJS build and no `require("tslog")` — v5 dropped dual publishing.
- **tsgo** (`@typescript/native-preview`, the TypeScript 7 native compiler) emits the ESM output (`dist/esm/`) and the declaration files (`dist/types/`).
- **esbuild** (`build.js`) bundles the browser IIFE (`dist/browser/index.js`, global `tslog`) from `src/index.browser.ts`.
- `"type": "module"` — the project is ESM throughout.
- `npm run build` = `clean-dist` (wipes `dist/` so stale files never ship) → `build-types` → `build-esm` → `build-browser` → `prepare-publish`.
- **Conditional exports** in `package.json` pick the entry per runtime: `node` → `index.node.js`, `browser`/`worker` → `index.browser.js`, `deno`/`bun`/`react-native`/`default` → `index.universal.js`. Subpaths (`tslog/transports/*`, `tslog/presets/*`, `tslog/otel`, `tslog/serializers`, `tslog/lite`, `tslog/slim`, `tslog/console`, `tslog/testing`, `tslog/pretty/box`, `tslog/throttle`, `tslog/cli`) are individually mapped and tree-shakeable (`sideEffects: false`, audited by `npm run audit-sideeffects`; gzip budgets for the slim/full browser bundles enforced by `npm run check-bundle-size`).
- The `tslog` bin (NDJSON pretty-printer) is `dist/esm/subpaths/cli.js`.

### Build configs

| Config | Purpose |
|---|---|
| `tsconfig.json` | Base (ES2022, strict, NodeNext) |
| `tsconfig.esm.json` | ESM output (`dist/esm/`) |
| `tsconfig.types.json` | Declaration files only (`dist/types/`) |

## Testing

- **Vitest** for Node.js and Bun tests
- **Playwright** for browser tests across **Chromium, Firefox, and WebKit** (via `playwright.config.ts`)
- **Deno.test** adapter (`tests/deno_runner.ts`) imports from `dist/esm/`
- Test files: `tests/*.test.ts` (numbered by feature area)
- Browser test files: `tests/*.browser.test.ts` — run only by Playwright, excluded from Vitest via the `tests/**/*.browser.test.ts` glob in `vitest.config.ts`
- Browser specs run against the IIFE bundle exposed as the global `window.tslog`; shared helpers in `tests/support/browser/browserHarness.ts` (`inPage` for logObj/return-value assertions, `captureConsole` for printed output) keep `page.evaluate` boilerplate out of the specs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fullstack-build/tslog](https://github.com/fullstack-build/tslog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
