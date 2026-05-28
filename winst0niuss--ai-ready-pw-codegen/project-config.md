---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

**AI-Ready PW Codegen** — offline Playwright recorder that captures user interactions with DOM snapshots, accessibility trees, and screenshots for later AI analysis (test generation, Page Object creation). Uses Playwright's built-in codegen as the UI/interaction layer. Acts as an "offline MCP Playwright" — record on a machine without AI access, then send the archive to Claude Code.

## Project Structure & Module Organization

Source files live in `src/`. `src/main.ts` is the CLI entry point, `src/recorder.ts` contains recorder orchestration, and `src/types.ts` defines shared interfaces. Browser snapshot logic is under `src/snapshot/`; reusable helpers are under `src/utils/`. Unit tests live in `src/__tests__/`. Public documentation copied into recording archives is in `docs/`, while compiled output goes to `dist/`. Generated recordings belong in `recordings/` and should not be committed.

## Commands

```bash
# Run the recorder (any of these are equivalent)
npx ts-node src/main.ts <URL> [options]
npm start -- <URL> [options]
npm run record -- <URL> [options]

# Options:
#   --no-screenshots     Disable screenshots
#   --no-archive         Skip .zip creation
#   --no-console         Disable console log capture
#   --no-network         Disable XHR/fetch network capture
#   --max-actions <N>    Stop after N actions
#   --output-dir <path>  Output directory (default: ./recordings)
#   --viewport-size=W,H  Viewport size (default: 1920,1080)
#   --jpeg [quality]     Override JPEG quality (JPEG is the default format, quality 80)

# Type check
npx tsc --noEmit

# Run all unit tests (Vitest)
npm test

# Run a single test file
npx vitest run src/__tests__/cli-parsers.test.ts   # URL/viewport parsing
npx vitest run src/__tests__/analysis-prompt.test.ts  # SESSION.md generation
npx vitest run src/__tests__/dom-cleaner.test.ts      # DOM cleaner (runs in jsdom)

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage

# Build to dist/
npm run build
```

## Coding Style & Naming Conventions

Use strict TypeScript targeting ES2022 and CommonJS. Prefer small modules organized by responsibility: CLI parsing in `utils`, browser evaluation code in `snapshot`, and recorder flow in `recorder.ts`. Use camelCase for functions and variables, PascalCase for exported types/interfaces, and kebab-case test files such as `cli-parsers.test.ts`. Preserve existing async/await patterns and graceful fallbacks for non-critical capture failures. Use `@ts-expect-error` only for documented Playwright internal APIs such as `_enableRecorder`.

## Testing Guidelines

Tests use Vitest, not Jest. Test files are discovered by `src/**/__tests__/**/*.test.ts`. The default test environment is `node`; add `// @vitest-environment jsdom` for DOM/browser API tests such as `dom-cleaner.test.ts`. Focus unit tests on pure utilities and snapshot helpers. Recorder and CLI changes often require manual verification with a live Playwright browser in addition to unit tests.

## Commit & Pull Request Guidelines

Git history follows Conventional Commit-style prefixes: `feat:`, `fix:`, `docs:`, and `chore:`. Keep commit subjects specific, for example `fix: handle write stream error in archiver`. Pull requests should describe the behavior change, list verification commands, link related issues, and include screenshots or sample recording output when user-visible recorder behavior changes.

## Architecture

Uses Playwright's built-in codegen (`context._enableRecorder()` internal API) for action capture and UI. AI-Ready PW Codegen hooks into codegen events to capture DOM snapshots, accessibility trees, screenshots, and console logs on each recorded action.

### Communication Protocol

```
Playwright Codegen (built-in recorder)
  → eventSink.actionAdded(page, data, code)
  → eventSink.actionUpdated(page, data, code)
    → recorder.ts enqueueAction → processAction (sequential Promise queue)
      → capture accessibility tree + cleaned DOM + screenshot + console logs
      → store in memory arrays (for actionUpdated overwrite support)
      → on finalize: write actions.jsonl + snapshots.jsonl to disk
```

**Dual `_enableRecorder` call**: First call opens the GUI inspector, second call (with `recorderMode: 'api'`) attaches the eventSink for programmatic access. Both coexist on the same context.

**Protocol auto-detection**: When URL has no protocol, tries `http://` first, falls back to `https://`. Explicit `http://` or `https://` used as-is.

**Important**: Uses Playwright internal API (underscore-prefixed). Playwright version is pinned (currently `1.59.1`) to prevent breakage — bump only after verifying `_enableRecorder` still works.

### Key Files

- **`src/main.ts`** — CLI entry point, URL validation with protocol fallback, launches Chromium, handles shutdown + archiving
- **`src/recorder.ts`** — Core class: enables codegen via `_enableRecorder`, listens for `actionAdded`/`actionUpdated` events, captures snapshots and console logs. Stores actions in memory arrays, writes JSONL on finalize. Supports `max-actions` stop via `onStop()` callback

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [winst0niuss/ai-ready-pw-codegen](https://github.com/winst0niuss/ai-ready-pw-codegen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
