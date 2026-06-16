---
trigger: always_on
description: A free Zotero 7, 8 & 9 plugin that shows citation counts, field-weighted impact, and journal rankings alongside the items in your library, and lets you follow citations forward and backward without leaving Zotero. Powered by [OpenAlex](https://openalex.org).
---

# Citegeist

A free Zotero 7, 8 & 9 plugin that shows citation counts, field-weighted impact, and journal rankings alongside the items in your library, and lets you follow citations forward and backward without leaving Zotero. Powered by [OpenAlex](https://openalex.org).

---

## Quick Reference

```bash
npm install                # Install dependencies (use this, not npm ci — see CI notes below)
npm run build:dev          # Dev build → build/addon (no minification)
npm run build              # Production build → build/citegeist-x.y.z.xpi
npm test                   # Run all tests (vitest)
npm run test:watch         # Re-run on file changes
npm run typecheck          # tsc --noEmit (strict)
npm run lint               # ESLint
npm run lint:fix           # ESLint --fix
npm run format             # Prettier write
npm run format:check       # Prettier check (no write)
npm run okf:check          # OKF docs-conformance (every docs/ file has a `type`)
npm run okf:drift          # Compare OKF spec upstream HEAD vs the pinned commit
npm run release            # Bump version, tag, push (triggers GitHub Actions release)
```

**Pre-commit checklist:** `npm run typecheck && npm test && npm run lint && npm run format:check && npm run okf:check && npm run build`

---

## Project Structure

```
src/
  index.ts                      # Bootstrap entry point
  constants.ts                  # All tunable constants (rate limits, timeouts, sizes)
  hooks.ts                      # Zotero lifecycle hooks
  modules/
    openalex.ts                 # OpenAlex API client — fetch, parse, rate limit, retry
    cache/                      # SQLite-backed cache (v2.0.0+)
      db.ts                     # Connection, in-memory mirror, lifecycle (init/close)
      read.ts                   # Sync read API (mirror only)
      write.ts                  # Async write API (SQLite first, then mirror)
      migration.ts              # One-shot Extra→SQLite migration + orphan GC
      types.ts                  # Public types + internal row shape + column list
      index.ts                  # Public surface (re-exports)
    citationService.ts          # Orchestration: fetch + cache + error handling
    citationColumn.ts           # Sortable item-tree columns
    citationPane.ts             # Item-detail sidebar pane
    menu.ts                     # Right-click context menus
    utils.ts                    # Shared: escapeHTML, normalizeError, logError, safeHTML
    citationNetwork/
      dialog.ts                 # Modal lifecycle (explicit DialogPhase state machine)
      results.ts                # Result rendering, pagination, infinite scroll
      actions.ts                # Add-to-library, undo, collection filing
      collectionPicker.ts       # Collection selection UI
      types.ts                  # Shared types, constants, DialogPhase enum
      styles.ts                 # CSS-in-JS for the dialog
      index.ts                  # Public API
    ui/                         # Canonical design system (CSS-in-JS, both surfaces)
      tokens.ts                 # cgDesignTokens — single source for --cg-* colour/space/type tokens
      components.ts             # cgComponents — shared primitives (.cg-btn/.cg-chip/.cg-card/.cg-banner/.cg-eyebrow)
      theme.ts                  # resolveHostScheme — forces color-scheme to Zotero's actual theme
  data/
    journalRankings.ts          # Static ISSN → ranking lookup (UTD24, FT50, ABDC, AJG)

test/                           # vitest unit tests
addon/                          # Static addon files (manifest.json, prefs.xhtml, icons)
scripts/                        # build.mjs, release helpers
tools/                          # okf-check.sh, okf-drift-check.sh (docs OKF standard)
typings/                        # Zotero type declarations
```

---

## Architecture Principles

**Error handling:** All caught errors flow through `normalizeError(e)` / `logError(context, e)` from `utils.ts`. Never use `"" + e` or template-string coercion — it drops stack traces.

**Network errors vs. 404:** `OpenAlexNetworkError` (from `utils.ts`) signals "service unreachable". A `null` return from `getWorkByDOI` signals "not found". UI layers check `result.error === "network"` to show the appropriate message.

**Constants:** Every magic number lives in `src/constants.ts`. Do not hardcode timeouts, sizes, or thresholds inline.

**Caching (v2.0.0+):** Cached metrics live in a plugin-owned SQLite database at `<profile>/citegeist.sqlite`. Reads hit a synchronous in-memory mirror loaded at startup (Zotero's column `dataProvider` is sync). Writes go to SQLite first, then update the mirror. Only the user-curated `Citegeist match ID: W…` line is mirrored back to Zotero's `Extra` field for downgrade safety and cross-device sync. See `src/modules/cache/` and `docs/MIGRATION-v2.0.0.md`. The one-shot migration from v1.3.x Extra-namespaced fields is in `cache/migration.ts`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phdemotions/zotero-citegeist](https://github.com/phdemotions/zotero-citegeist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
