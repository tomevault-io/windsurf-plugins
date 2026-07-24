---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SEOmator is a comprehensive SEO audit tool (`@seomator/seo-audit`) with 251 rules across 20 categories. It ships as both a **CLI tool** (published to npm) and an **Electron desktop app** (local only). It fetches web pages, parses HTML with Cheerio, optionally measures Core Web Vitals via Playwright, and scores pages against SEO best practices.

## Critical Rules (read before making changes)

### package.json Dual-Purpose Constraints

The `package.json` serves **both** the npm CLI package and the Electron desktop app. These fields have strict requirements:

| Field | Value | Why |
|-------|-------|-----|
| `main` | `./dist-electron/main/index.js` | **Electron reads this** to find the main process entry. DO NOT change to `./dist/cli.js` or Electron will execute Commander CLI instead of launching the app window. |
| `exports` | `./dist/cli.js` | **npm/Node.js consumers use this** for programmatic imports. Takes priority over `main` in modern Node.js. |
| `bin` | `./dist/cli.js` | **npm CLI users use this** (`seomator` command). |
| `files` | `["dist"]` | **Only `dist/` ships to npm.** This is the firewall — `electron/`, `dist-electron/`, `scripts/` never reach npm users. |

**If you change `main` to anything other than the Electron entry, `npm run electron:dev` will break** — Electron will print Commander help text and exit instead of opening the app window.

### Dependency Split: CLI vs Electron

- **`dependencies`**: Only CLI packages (cheerio, commander, playwright, better-sqlite3, etc.). These are what npm users install.
- **`devDependencies`**: Electron-only packages (react, react-dom, react-router-dom, recharts, zustand, electron, electron-vite, tailwindcss, etc.).
- **Never move react/zustand/recharts/electron packages into `dependencies`** — npm users would download ~15MB of unused Electron UI code.

### npm Publishing Checklist

1. Verify `dependencies` contains only CLI packages (no react, zustand, recharts, electron)
2. Verify `files: ["dist"]` — only CLI build ships
3. `npm run build` → builds CLI via tsup
4. `npm pack --dry-run` → confirm only `dist/` files + README + package.json
5. `npm publish --access public`
6. The `prepublishOnly` script auto-runs `npm run build` before publish

Published as `@seomator/seo-audit` on npm. Current version: **3.0.0**.

### better-sqlite3 Native Module ABI

`better-sqlite3` is a C++ addon compiled against a specific Node.js ABI. Electron and CLI/Node.js use different ABIs:

```bash
npx electron-rebuild -f -w better-sqlite3   # Before running Electron
npm rebuild better-sqlite3                    # Before running CLI tests
```

**You must recompile when switching between Electron and CLI.** Failure produces a cryptic `NODE_MODULE_VERSION mismatch` error.

### Zero Modifications to `src/`

The Electron app is **purely additive** — all Electron code lives in `electron/`. The `src/` directory is shared by both CLI and Electron through direct imports (via the `@core` alias in electron-vite config). Never put Electron-specific code in `src/`.

## Build & Development Commands

### CLI

```bash
npm run build          # Build with tsup (ESM, single entry: src/cli.ts)
npm run dev            # Build in watch mode
npm run test:run       # Run all tests once (vitest)
npm test               # Run tests in watch mode
```

Run locally after building:
```bash
./dist/cli.js audit https://example.com --no-cwv
```

Run a single test file:
```bash
npx vitest run src/rules/core/core.test.ts
```

### Electron Desktop App

```bash
npx electron-rebuild -f -w better-sqlite3   # Required before first run
npm run electron:dev   # Dev mode with Vite HMR + Electron hot reload
npm run electron:build # Production build (main + preload + renderer)
npm run electron:pack  # Build + package into distributable
```

## Architecture

### Rule System (the core abstraction)

The entire audit engine is built on a **self-registering rule pattern**:

1. **`defineRule()`** (`src/rules/define-rule.ts`) - Creates and validates an `AuditRule` object with `id`, `name`, `description`, `category`, `weight`, and `run(context)` function.

2. **`registerRule()`** (`src/rules/registry.ts`) - Stores rules in a global `Map<string, AuditRule>`. Throws on duplicate IDs.

3. **Category `index.ts` files** (e.g., `src/rules/core/index.ts`) - Import individual rule files and call `registerRule()` for each. This is the registration point.

4. **`src/rules/loader.ts`** - Static-imports all 20 category `index.ts` files. The act of importing triggers side-effect registration. `loadAllRules()` exists for API compat but rules load at import time.

5. **Result helpers**: `pass(ruleId, message, details?)`, `warn(...)`, `fail(...)` return `RuleResult` with scores 100/50/0 respectively.

### Adding a New Rule

1. Create `src/rules/<category>/<rule-name>.ts` exporting a const created via `defineRule()`
2. Rule ID convention: `<category>-<descriptive-name>` (e.g., `core-canonical-conflicting`)
3. Import and `registerRule()` in `src/rules/<category>/index.ts`
4. The `run` function receives `AuditContext` and returns `RuleResult` (or Promise)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seo-skills/seo-audit-skill](https://github.com/seo-skills/seo-audit-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
