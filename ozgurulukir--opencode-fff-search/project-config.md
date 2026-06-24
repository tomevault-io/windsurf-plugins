---
trigger: always_on
description: This document provides essential context for AI agents working on the opencode-fff-search codebase.
---

# AGENTS.md

This document provides essential context for AI agents working on the opencode-fff-search codebase.

## Project Overview

Plugin that replaces the built-in `grep` and `glob` file search tools with [fff](https://github.com/dmtrKovalenko/fff)'s ultra-fast, typo-resistant search engine. Supports both OpenCode and MiMo Code.

**Key characteristics:**

- Modular ES module plugin (6 files: `index.js`, `search.js`, `helpers.js`, `filters.js`, `gitignore.js`, `constants.js`)
- No build step required
- Node.js 18+ required (ES modules)
- Uses `@ff-labs/fff-node` (Node.js) or `@ff-labs/fff-bun` (Bun runtime) with auto-detection, plus `minimatch` for glob matching
- Returns `{ output, metadata }` objects so OpenCode's TUI renders match counts inline
- **Full feature set** — grep (pattern, path, include, exclude, caseSensitive, context, limit), glob (pattern, path, type, limit)
- **Single-file 100% recall** — When path points to a file, reads it directly bypassing fff index
- **aiMode enabled** — Frecency scoring on by default for better recall and ranking
- **Smart mode detection** — Detects regex vs plain patterns; plain uses SIMD-accelerated literal matching
- **Dual platform** — Supports both OpenCode (`@opencode-ai/plugin`) and MiMo Code (`@mimo-ai/plugin`) with runtime auto-detection

## Architecture

### Plugin Structure

The plugin exports an async default function `(input)` (also exported as `server` for MiMo Code compatibility) that:

1. **Lazily imports** fff-native (`@ff-labs/fff-bun` or `@ff-labs/fff-node` via `lazyFff()`) at runtime — never at module level, avoiding Bun-on-Windows module-graph crashes
2. **Auto-detects** plugin SDK (`@mimo-ai/plugin` or `@opencode-ai/plugin`) at runtime
3. **Initializes** a `FileFinder` instance with safe defaults; gracefully falls back to fs-only mode when `FileFinder.create()` returns `ok: false`
4. **Caches** one `FileFinder` per directory (module-level `instances` Map) to prevent native resource leaks
5. **Creates a shared `scanPromise`** to avoid multiple concurrent index scans
6. **Returns tool definitions** that override OpenCode's built-in `grep` and `glob` tools

### Data Flow

```
grep:
  File path      → directFileGrep (Node.js fsPromises.readFile) → format
  Unicode pattern → fsGrep (fsPromises.readdir + fsPromises.readFile + Unicode regex) → post-filter → format
  ASCII pattern  → fff grep (plain or regex mode) → if zero → plain→regex retry → fsGrep fallback → post-filter → format

glob:
  Metachar + type=directory → globWalk directly (fff directorySearch is fuzzy, not glob-aware)
  Metachar + type=file      → fff fileSearch → minimatch post-filter → globWalk fallback → absolute paths → format
  Fuzzy query               → fff fileSearch/directorySearch → filter by path → globWalk fallback → absolute paths → format
  Fuzzy query (no exact match) → fff fileSearch + globWalk augmentation → absolute paths → format
```

### Tool Output Format

- **grep tool**: Returns `{ title, output: string, metadata: { matches: number, truncated: boolean } }`
  - Output format: `relativePath:lineNumber:lineContent` (one line per match)
  - When `context > 0`: renders `contextBefore` lines, match line, `contextAfter` lines with correct line numbers
  - Default limit: 100 matches, configurable 1–5000
- **glob tool**: Returns `{ title, output: string, metadata: { count: number, truncated: boolean } }`
  - Output format: newline-separated absolute file paths
  - Default limit: 100 results, configurable 1–5000

### Key Components

- `lazyFff(client)` — Lazily imports `@ff-labs/fff-bun` (Bun) or `@ff-labs/fff-node` (Node.js) at runtime, avoiding Bun-on-Windows module-graph crashes. Safe to call multiple times (idempotent).
- `FileFinder.create({ basePath, ...config })` — Initializes fff search engine with aiMode enabled; gracefully falls back to fs-only mode when `ok: false`
- `instances` Map — Module-level cache: one `{ finder, scanPromise }` per directory
- `finder.waitForScan(15000)` — Waits for initial index build (15s timeout)
- `detectGrepMode(pattern)` — Returns `"regex"` or `"plain"` based on regex metachar detection
- `finder.grep(pattern, opts)` — Content search with regex/plain mode + smart case + cursor pagination
- `directFileGrep(filePath, basePath, pattern, ctxLines)` — Direct file read for 100% recall on single-file searches
- `fsGrep(dir, basePath, pattern, ctxLines, pathFilter, include, exclude)` — Directory-level grep for non-ASCII (Unicode/Turkish) patterns; walks dirs with `fsPromises.readdir` and reads files with `fsPromises.readFile` using exact Unicode regex (`u` flag). Bypasses fff's Unicode normalization to avoid `ş↔s` overcount. Applies include/exclude during traversal. Include/exclude patterns are pre-parsed once via `parsePatterns()` before the loop.
- `globWalk(dir, pattern, basePath, limit, type)` — Real glob matching via recursive `fsPromises.readdir` + minimatch (supports file/directory type)
- `loadGitignoreFilter(basePath)` — Async function that reads `.gitignore` via `fsPromises.readFile` and augments `SKIP_DIRS` with directory-name entries; cached per basePath. Used by `fsGrep` and `globWalk` (both `await` it).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ozgurulukir/opencode-fff-search](https://github.com/ozgurulukir/opencode-fff-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
