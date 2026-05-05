---
trigger: always_on
description: - Purpose: guidance for agentic coding assistants working in this repository.
---

# AGENTS for Get笔记 Importer (root scope)
- Purpose: guidance for agentic coding assistants working in this repository.
- Scope: applies to entire repo until overridden by a deeper AGENTS.md (none exist currently).
- Audience: agents making code changes; follow when reading/writing any file here.
- Keep edits minimal, targeted, and consistent with existing patterns.
- When instructions conflict, precedence is system > user > developer > this file.

## Quick project facts
- Plugin name: Get笔记 Importer (Get笔记 to Obsidian).
- Entry point: `main.ts`; bundled output: `main.js`.
- Core logic: `lib/get/*` (auth, exporter, core, importer).
- UI components: `lib/ui/*`; visuals: `lib/obIntegration/*`, `styles.css`.
- Hard-coded cache paths live in `lib/get/const.ts`.
- Incremental-sync IDs defined in `lib/get/core.ts`—avoid breaking format.
- Attachments default path: `get attachment/` under chosen vault folder.
- Target folders default to `get/notes`, configurable via settings.
- No tests or CI present; verify manually.
- Playwright desktop-only dependency; version locked.

## Setup commands
- Install deps: `npm install` (node 16+ recommended).
- Install Playwright (required): `npx playwright@1.43.1 install`.
- Optional: `npm run dev` for watch build (esbuild + sourcemaps).
- Ensure Obsidian desktop available for runtime validation.

## Build / lint / format commands
- Fast dev build: `npm run dev` (rebuilds to `main.js`).
- Prod build: `npm run build` (tsc noEmit + esbuild production bundle).
- Type-only compile: `npm run compile` (tsc emit per tsconfig).
- Lint check: `npm run lint` (gts / ESLint Google TS style).
- Auto-fix + format: `npm run fix` (gts fix, includes prettier rules).
- Clean generated artifacts: `npm run clean` (gts clean).
- Deploy script: `npm run deploy` (build then `./deploy.sh`).
- Version bump helper: `npm run version` (updates manifest/versions.json).
- There is no `test` script; see test section below.

## Testing status
- No automated tests exist; `npm test` is undefined.
- Single-test execution is unavailable because no test runner/config exists.
- If manual verification needed, run build then load plugin in Obsidian.
- Do not add new test frameworks unless explicitly requested.

## Runtime constraints and cautions
- Do not modify memo ID generation in `lib/get/core.ts` without approval; it underpins incremental sync.
- Preserve hard-coded cache paths in `lib/get/const.ts` unless task requires change.
- Playwright flows assume desktop; avoid adding mobile-only features.
- Attachment path assumptions exist in importer/exporter; keep consistent with settings.
- Background sync uses timers; ensure intervals cleared on unload to prevent leaks.
- Avoid introducing new global state; rely on plugin settings storage (`saveData`).

## TypeScript / lint baseline
- Codebase follows `gts` (Google TypeScript Style) via ESLint/Prettier.
- Use semicolons; prefer single quotes; 2-space indents (respect local file indentation where tabs already exist).
- Keep lines ≤ 100 chars when practical; wrap long strings thoughtfully.
- Enable strictness manually only if task requires; `tsconfig` sets `strict: false` and `skipLibCheck: true`.
- Target ES2022, module `commonjs`, `esModuleInterop: true`; avoid ESM-only syntax.
- Avoid unused imports/vars; lint will flag.

## Imports
- Order: node/third-party modules first, then absolute aliases (none), then relative paths.
- Group and separate categories with blank lines only when already present.
- Use named imports when available; default imports only when module exports default (e.g., `turndown`).
- Prefer `import type` for type-only references to keep runtime bundle small.
- Avoid deep relative traversals when existing barrel exports suffice (none currently).
- Keep Obsidian imports explicit (`Plugin`, `Notice`, etc.).

## Formatting
- Run `npm run fix` before handing off if time allows; it applies ESLint+Prettier.
- Maintain existing indentation style in touched files to avoid noisy diffs (tabs in `main.ts`, 4-space blocks elsewhere).
- Use trailing commas in multi-line objects/arrays when formatter expects.
- Place braces on same line; always brace conditional/loop bodies.
- Keep blank lines minimal; preserve logical grouping around sections.

## Types and interfaces
- Prefer explicit return types on exported functions/methods.
- Use interfaces for data shapes (`GetNote`, memo objects) rather than type aliases when extendable.
- Avoid `any`; use `unknown` with narrowing if necessary.
- Favor `readonly` for configuration objects that should not mutate.
- Use `string | undefined` instead of nullable magic values; default via parameters.
- Narrow HTML/DOM query results before use; check for nulls in Obsidian DOM APIs.

## Naming conventions
- Classes: PascalCase (`Get笔记Importer`, `Get笔记Core`).
- Functions/methods: camelCase; booleans read as predicates (`isAuthFileExist`, `mergeByDate`).
- Constants: SCREAMING_SNAKE for module-level immutables (`AUTH_FILE`, `DOWNLOAD_FILE`).
- Private helpers: prefix with `_` only if lint permits; otherwise keep camelCase.
- Avoid single-letter variables except for small loops.

## Async / promises
- Prefer `async/await`; avoid raw promise chains.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geekhuashan/get-to-obsidian](https://github.com/geekhuashan/get-to-obsidian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
