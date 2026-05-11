---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Obsidian plugin that automatically updates frontmatter `created` and `updated` timestamps when files are edited.

**Target: Obsidian v1.4.11+** (`minAppVersion` in manifest.json).

## Obsidian API — IMPORTANT

Claude's training knowledge about the Obsidian plugin API is outdated. **Always** use context7 MCP and official Obsidian developer documentation to verify API signatures, available methods, and current patterns before writing or modifying plugin code. Do not rely on memory — check the docs.

Key Obsidian API used in this plugin:
- `Plugin` lifecycle: `onload()`, `onunload()`
- Vault events: `app.vault.on('modify' | 'create' | 'rename' | 'delete', ...)`
- Frontmatter: `app.fileManager.processFrontMatter(file, callback, options)`
- Data persistence: `Plugin.loadData()` / `Plugin.saveData()`
- UI: `PluginSettingTab`, `Setting`, `Modal`, `Notice`, `AbstractInputSuggest`
- Status bar: `Plugin.addStatusBarItem()`

## Build & Dev Commands

```bash
make                  # Show all available commands
make install          # Install dependencies (npm ci)
make build            # Type-check + production build → dist/
make dev              # Dev mode with watch (esbuild + CSS + optional vault sync)
make lint             # ESLint with eslint-plugin-obsidianmd (matches community review bot)
make format-check     # Prettier check
make format           # Prettier fix
make test             # Run all tests (vitest)
make test-watch       # Run tests in watch mode
make pre-commit       # Run all checks: format, lint, test, build
make local-test       # Build and copy plugin to local Obsidian vault
```

Run a single test file: `npm test -- src/__tests__/formatDate.test.ts`

### Local vault testing

Set `OBSIDIAN_VAULT` env var (or `.env` file, see `.env.example`) to your vault path. `make local-test` builds the plugin and copies artifacts to `<vault>/.obsidian/plugins/frontmatter-date-manager/`. `make dev` uses the same variable for auto-copy on file changes.

## Build Output

esbuild bundles `src/main.ts` → `dist/main.js` (CJS, ES2018 target). `manifest.json` is copied to `dist/`. CSS from `styles.css` is processed with lightningcss → `dist/styles.css`. The `__DEV_MODE__` global is `true` in dev, `false` in production (controls console logging).

## Architecture

- **`src/main.ts`** — `FrontmatterDateManagerPlugin` (extends `Plugin`). Entry point. Handles vault events (`modify`, `create`, `rename`, `delete`). Per-file debouncing (2s). SHA-256 content hashing to detect real changes. Hash cache persisted to `hash-cache.json` with LRU eviction, debounced flushing, GC on startup, auto-population, and format migration. Pause/resume with countdown timer and status bar indicator. Three commands: `update-timestamps-current-file`, `toggle-auto-update`, `pause-auto-update`. Uses `processingFiles` Set to prevent concurrent modifications of the same file. `log()` and `logError()` are public methods gated behind `__DEV_MODE__` — used by modals and other files for dev-only logging.
- **`src/filterRules.ts`** — Gitignore-style filter rule engine. Pure functions, no Obsidian dependency. `parseFilterRules(text)` parses multiline text into `FilterRule[]` with validation errors. `isFileExcluded(filePath, rules)` applies rules top-to-bottom (last match wins).
- **`src/Settings.ts`** — `FrontmatterDateManagerSettings` interface + `FrontmatterDateManagerSettingsTab` (settings UI). `HashTrackingMode` type (`'body'` | `'frontmatter'` | `'both'`). Manages date format, timezone, frontmatter key names, gitignore-style filter rules (textarea with preview), min time between saves, delay for new files, number property toggle, content hash check toggle, hash tracking mode dropdown, frontmatter key exclusion list, hash cache settings, post-update command.
- **`src/BaseBulkModal.ts`** — Abstract base class for bulk file operations. Provides progress bar, error counting, Run/Cancel UI. Extended by `UpdateAllModal` and `UpdateAllCacheData`.
- **`src/UpdateAllModal.ts`** — Modal for bulk-updating all vault files' timestamps.
- **`src/UpdateAllCacheData.ts`** — Modal for populating/rebuilding SHA-256 hash cache for all files.
- **`src/BulkPopulateTimestampsModal.ts`** — Wizard modal for bulk-populating frontmatter timestamps from filesystem dates (`ctime`/`mtime`). Three-phase flow: configure (mode/override selection + platform warnings) → preview (dry-run with scrollable table) → execute (progress bar). Supports `PopulateMode` (`'created'`|`'updated'`|`'both'`) and `OverrideMode` (`'fill-missing'`|`'overwrite-all'`). Uses `Platform.*` API for platform-specific ctime reliability warnings. Does NOT extend `BaseBulkModal` (different wizard-based UI flow).
- **`src/RenameKeyModal.ts`** — Wizard modal for renaming frontmatter keys across all files. Three-phase flow: configure (old/new key names + delete toggle) → preview (scrollable table with conflict detection) → execute (processFrontMatter rename + optional delete). Scans ALL markdown files (`vault.getMarkdownFiles()`), not just filter-eligible ones. Does NOT extend `BaseBulkModal`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SmetDenis/obsidian-frontmatter-date-manager](https://github.com/SmetDenis/obsidian-frontmatter-date-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
