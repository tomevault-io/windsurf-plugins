---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**neostandard** is a modern successor to `standard` — an ESLint **shareable config** distributed as a flat-config (ESLint 9/10) array. It is plain JavaScript (ESM, `"type": "module"`) typed with JSDoc; `tsc` only type-checks and emits `.d.ts` declarations — there is **no compile step**, the `.js` you edit is the `.js` that ships. CommonJS consumers are served via require(esm): `index.js` has an `export { … as 'module.exports' }` interop export, so `require('neostandard')` returns the callable with `plugins`/`resolveIgnoresFromGitignore` attached as 0.13-compat properties (they are also real ESM named exports — the primary API).

It replaces a family of separate packages with one configurable function:
- `ts: true` → what `ts-standard` did
- `semi: true` → what `semistandard` did
- `noStyle: true` → drops all formatting rules (pair with Prettier/dprint)
- `noJsx: true` → drops JSX parsing and JSX style rules (React logic rules from `eslint-plugin-react` are currently absent pending ESLint 10 support — see issue #350)

It sits at the **base of a dependency stack** (`eslint → @voxpelli/eslint-config → neostandard → @stylistic`). Consumers inherit its choices, so **stability governs version ranges over novelty** (see the @stylistic pin below).

## Architecture

`neostandard(options)` returns an `import('eslint').Linter.Config[]` — a flat-config **array**, not a single object. Tracing the data flow matters because order and composition are deliberate:

- **`index.js`** — public surface. **Named exports are the primary API**: `neostandard`, `resolveIgnoresFromGitignore`, `plugins`. The default export is the same function with the other two attached as properties (0.13 compat — `neostandard.resolveIgnoresFromGitignore()` and the `require()` flavor are used in the wild; harmless in ESM, keep them). `plugins` uses **lazy getters** (`@stylistic`, `n`, `promise`, `neostandard`, `typescript-eslint`) so consumers can layer extra config using the *exact same plugin instances* neostandard uses, without eager loading cost. The getters (and the lazy jsx/style/ts layer loads in `lib/main.js`) go through `createRequire` + require(esm), which shares the ESM module registry — including `lib/tseslint.js`, a tiny indirection that pins the getter to the ESM build of the dual-published `typescript-eslint` (a bare `require` would grab its CJS build and break instance identity). (`react` is omitted while `eslint-plugin-react` is incompatible with ESLint 10 — issue #350; the one reimplemented rule is exposed via the `neostandard` getter — see `lib/plugin/` below.)
- **`lib/main.js`** — the composer. Resolves `env`/`globals` (via the `globals` package), computes the positive **file scopes**, then assembles the array in this order: lone-key `{ ignores }` first (must be alone to count as a *global* ignore per ESLint), then `neostandard/globals`, then the JS layers, then a single TypeScript block last. Throws if `filesTs` is passed without `ts`. **Every rule-bearing layer carries a positive `files` scope** so neostandard only ever touches the JS/TS files it owns and never leaks onto other languages a consumer layers on (`@eslint/markdown`, `@eslint/json`, …) — this is the #296 fix. (The lone-key `{ ignores }` block is the exception: ESLint requires it to have no other keys to count as a *global* ignore, so it carries neither `files` nor the md-block `ignores`.) The scopes: the core rule layers + `globals` get `jsTsFiles` (`**/*.{js,cjs,mjs,jsx}` + user `files`, plus `**/*.{ts,tsx}` + `filesTs` when `ts`) because the TS block only *adds overrides* (see `lib/ts.js`) and relies on these layers for the actual ruleset; the `jsx`/`style/jsx` layers get the narrower `jsxFiles` (`**/*.jsx`, `**/*.tsx`, + user `files`); the TS block gets `tsFiles`. Every rule-bearing layer also carries `ignores: ['**/*.md/**']` so neostandard's rules stay off Markdown fenced code blocks (the virtual `*.md/*.js` files from `@eslint/markdown`'s processor).
- **`lib/configs/*`** — each file is one (or two) complete flat-config object(s), composed as layers:
  - `base.js` — core best-practice rules + `n` and `promise` plugins (always on)
  - `modernization.js` — exports `modernization` (relaxations since standard 17, e.g. `n/no-deprecated-api` → warn, `dot-notation` relaxed) and `modernizationStyles` (always on; style-only relaxations gated by `noStyle`)
  - `style.js` — `@stylistic` rules; default is **no semicolons**
  - `semi.js` — flips `@stylistic/semi` to `always`; included only when `semi: true`
  - `jsx.js` — exports `jsx` (JSX parsing via `ecmaFeatures.jsx`, plus the `neostandard/jsx-key` rule via the internal plugin) and `jsxStyles` (`@stylistic` JSX formatting rules); included unless `noJsx`. React-specific logic rules (`eslint-plugin-react`) are currently absent pending ESLint 10 support (issue #350)
  - `ts-redundant.js` — map of ~20 core rules set to `off` because TypeScript already enforces them

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neostandard/neostandard](https://github.com/neostandard/neostandard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
