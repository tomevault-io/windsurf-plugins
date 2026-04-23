---
trigger: always_on
description: `@cloudflare/telescope` is a TypeScript browser performance testing library and CLI built on Playwright. It launches real browsers (Chrome, Chrome Beta, Canary, Firefox, Safari, Edge), collects HAR files, Web Vitals, and performance metrics, and produces HTML reports.
---

# Telescope — Agent Guide

## Repository Overview

`@cloudflare/telescope` is a TypeScript browser performance testing library and CLI built on Playwright. It launches real browsers (Chrome, Chrome Beta, Canary, Firefox, Safari, Edge), collects HAR files, Web Vitals, and performance metrics, and produces HTML reports.

Key subdirectories:

- `src/` — TypeScript source compiled to `dist/`
- `__tests__/` — Vitest integration tests (excluded from tsconfig)
- `tests/` — Static test fixtures (HTML, CSS, images) used by integration tests
- `support/` — Browser support files (e.g., Firefox default `user.js` preferences)
- `processors/` — Standalone post-processing report generator (included in main tsconfig)
- `telescopetest-io/` — Separate Astro + Cloudflare Workers web app (fully excluded from root tooling)

---

## Build, Lint, and Test Commands

### Build

```bash
npm run build          # tsc + copy templates to dist/
npm run dev            # tsc --watch
```

**Tests require a build first.** Some tests invoke `node dist/src/cli.js` via `spawnSync`, so they test compiled output.

### Lint

```bash
npm run lint           # eslint .
npm run lint:fix       # eslint . --fix
npm run prettier       # npx prettier --write .
```

### Test

```bash
npm test               # build + vitest run
npm run test:ci        # export CI=true; npm test
npm run coverage       # vitest run --coverage
```

**Run a single test file** (build first):

```bash
npm run build && npx vitest run __tests__/cli.test.ts
```

**Run a single test by name:**

```bash
npm run build && npx vitest run __tests__/cli.test.ts -t "generates a Har file"
```

**Important:** `maxWorkers: 1` is required — tests launch real browsers and cannot run in parallel. Tests that call `launchTest()` or spawn the CLI directly need explicit timeouts (60000–120000ms). See `vitest.config.ts` for configuration.

---

## TypeScript Configuration

- **Target**: ES2022, **Module**: NodeNext, **ModuleResolution**: NodeNext
- **Strict mode**: fully enabled (`strict: true`)
- Additional strict flags: `noUnusedLocals`, `noUnusedParameters`, `noImplicitReturns`, `noFallthroughCasesInSwitch`
- **All local imports must use `.js` extensions** even when importing `.ts` source files — required by `module: NodeNext`

```typescript
// Correct
import { log } from './helpers.js';
import type { LaunchOptions } from './types.js';

// Wrong — will fail to resolve
import { log } from './helpers';
```

---

## Code Style Guidelines

### Imports

- Use **named imports** as the default; use default imports only for packages that export a default (e.g., `playwright`, `path`, `ejs`)
- Use **`import type { ... }`** for all type-only imports — enforced by ESLint (`@typescript-eslint/consistent-type-imports: error`)
- No path aliases — use relative paths (`./types.js`, `../src/index.js`)
- Group: external packages first, then internal relative imports

```typescript
import { Command, Option } from 'commander';
import playwright from 'playwright';

import type { BrowserContext } from 'playwright';
import type { LaunchOptions, TestResult } from './types.js';
import { log, generateTestID } from './helpers.js';
```

### Formatting (Prettier)

- **Single quotes** for strings
- **2-space indentation**
- **Trailing commas** everywhere (arrays, objects, function parameters)
- **No parentheses** on single-argument arrow functions: `x => x * 2` not `(x) => x * 2`
- EJS templates formatted via `prettier-plugin-ejs`

### Naming Conventions

| Entity                   | Convention         | Example                                                    |
| ------------------------ | ------------------ | ---------------------------------------------------------- |
| Files                    | `camelCase.ts`     | `testRunner.ts`, `defaultOptions.ts`                       |
| Classes                  | `PascalCase`       | `TestRunner`, `BrowserConfig`, `ChromeRunner`              |
| Interfaces               | `PascalCase`       | `LaunchOptions`, `NetworkProfile`, `TestPaths`             |
| Type aliases             | `PascalCase`       | `BrowserName`, `ConnectionType`, `TestResult`              |
| Functions                | `camelCase`        | `launchTest()`, `normalizeCLIConfig()`, `generateTestID()` |
| Constants                | `UPPER_SNAKE_CASE` | `DEFAULT_OPTIONS`                                          |
| Class methods/properties | `camelCase`        | `setupTest()`, `browserConfig`, `consoleMessages`          |
| Unused parameters        | prefix with `_`    | `_cleanupError`, `_unusedArg`                              |

### Types

- **All shared types** live in `src/types.ts` — the single source of truth. Add new types there.
- Use **`interface`** for objects with multiple properties; **`type`** for unions, primitives, and derived types
- Use `Record<K, V>` for maps, `Partial<T>` for optional shapes, `Pick<T, K>` for property subsets
- Use utility types to derive rather than duplicate: `Exclude<ConnectionType, false>`, `Parameters<BrowserContext['addCookies']>[0][number]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/telescope](https://github.com/cloudflare/telescope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
