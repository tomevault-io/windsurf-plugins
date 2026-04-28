---
trigger: always_on
description: Twitter/X topic selector skill for AI agents. Scrapes tweets via Chrome CDP, scores them (data metrics or Gemini AI), generates Markdown topic recommendation reports. This is a **Bun-first TypeScript project** — no bundler, no framework, scripts run directly with `bun run`.
---

# AGENTS.md — x-ai-topic-selector

## Project Overview

Twitter/X topic selector skill for AI agents. Scrapes tweets via Chrome CDP, scores them (data metrics or Gemini AI), generates Markdown topic recommendation reports. This is a **Bun-first TypeScript project** — no bundler, no framework, scripts run directly with `bun run`.

## Build / Lint / Test Commands

```bash
# Type-check (no emit — tsconfig has "noEmit": true)
bunx tsc --noEmit

# Run unit tests (61 tests across 3 files)
bun test

# Run main script (requires Chrome + X login)
bun run scripts/x-topic-selector.ts <source-url> [options]

# Example: dry-run with 10 tweets from a list
bun run scripts/x-topic-selector.ts 1234567890 --dry-run --max-tweets 10

# Example: AI scoring mode
bun run scripts/x-topic-selector.ts "https://x.com/i/lists/123" --score-mode ai-only --top-n 5
```

**Test Suite**: Bun built-in test runner with 61 test cases covering:
- 17 pure functions (data scoring, filtering, keyword extraction, report formatting)
- 6 API integration functions (AIClient mock-based testing via `MockAIClient`)
- Test files located in `tests/` directory
- No E2E tests for Chrome CDP (explicitly excluded)

**No linter or formatter is configured.** Follow the implicit conventions documented below.

## File Structure

```
scripts/
  x-topic-selector.ts   # Main entry — CLI parsing, Chrome launch, tweet scraping, orchestration
  ai-client.ts           # AI client abstraction — AIClient interface, GeminiClient, OpenAICompatibleClient, factory
  ai-scorer.ts           # AI scoring logic — batch scoring with concurrency control
  report-generator.ts    # Markdown report generation — engagement stats, keyword charts, recommendations
  x-utils.ts             # Chrome CDP connection, port management, platform-specific Chrome discovery
tests/
  x-topic-selector.test.ts   # Unit tests for main entry functions (20 tests)
  ai-scorer.test.ts          # Unit tests for AI scoring functions (18 tests)
  report-generator.test.ts   # Unit tests for report generation (21 tests)
SKILL.md                 # Agent interaction definition (question flows, parameter mapping)
output/                  # Generated reports (gitignored)
```

## TypeScript Configuration

- **Target**: ES2020, **Module**: ESNext, **Resolution**: bundler
- **Strict mode**: enabled
- **No emit**: scripts are run directly via `bun run`, not compiled
- Imports use `.js` extensions (Bun resolves `.ts` → `.js` at runtime)

## Code Style

### Imports

- Use `node:` protocol for Node.js builtins: `import fs from 'node:fs'`, `import { spawn } from 'node:child_process'`
- Use `.js` extensions for local imports: `import { sleep } from './x-utils.js'`
- Group: node builtins first, then local modules
- Use named imports; default imports only for Node builtins (`import fs from 'node:fs'`)
- Type-only imports use `import type`: `import type { AIScoredTweet } from './ai-scorer.js'`

### Formatting

- 2-space indentation
- Single quotes for strings
- Semicolons required
- Trailing commas in multi-line constructs
- ~100 char line width (soft limit, no enforced formatter)
- Template literals for string interpolation and multi-line strings

### Naming Conventions

- **Interfaces**: PascalCase (`Tweet`, `ScoredTweet`, `TopicSelectorOptions`)
- **Types**: PascalCase (`SourceType`, `PlatformCandidates`)
- **Functions**: camelCase (`calculateDataScore`, `filterAndScoreTweets`)
- **Constants**: UPPER_SNAKE_CASE for module-level config (`THREAD_EXPANSION_CONCURRENCY`, `DEFAULT_BATCH_SIZE`, `SELECTORS`)
- **Variables**: camelCase
- **Files**: kebab-case (`x-topic-selector.ts`, `ai-scorer.ts`)
- Prefix `is`/`has` for booleans: `isRetweet`, `isThread`, `hasEnglishContent`

### Types

- Prefer `interface` for object shapes, `type` for unions/aliases
- Use `as const` for readonly literal objects (see `SELECTORS`)
- Use generics on CDP `send<T>()` calls for return type safety
- Avoid `any` — use `unknown` and narrow. The one exception: `as any` appears minimally in scoring bridge code
- Optional fields use `?`: `isArticle?: boolean`

### Functions

- `async`/`await` throughout — no raw Promise chains
- Export only public API functions; keep helpers module-private
- Use arrow functions for callbacks/inline, `function` keyword for named top-level functions
- Main entry pattern: `async function main(): Promise<void>` with top-level `await main().catch(...)`

### Error Handling

- Try/catch with specific error messages: `throw new Error('Chrome not found. Set X_BROWSER_CHROME_PATH env var.')`
- Graceful degradation: AI scoring failures return default scores, don't crash
- Console logging with module prefix: `console.log('[x-topic-selector] ...')`, `console.warn('[ai-scorer] ...')`
- Empty catch blocks only for cleanup code (Chrome process kill, WebSocket close)
- Pattern: `error instanceof Error ? error.message : String(error)`

### Chrome CDP Patterns

- Use `CdpConnection` class from `x-utils.ts` for all WebSocket communication
- Always clean up: close targets in `finally` blocks, kill Chrome process on exit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vigorX777/x-ai-topic-selector](https://github.com/vigorX777/x-ai-topic-selector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
