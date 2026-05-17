---
trigger: always_on
description: npm run build        # → .scaffold/build/zo-files.xpi
---

# CLAUDE.md — Development Guide for ZoFiles

## Quick Start

```bash
npm install
npm run build        # → .scaffold/build/zo-files.xpi
npm start            # dev mode with hot reload
```

## What Is This

**ZoFiles** is a Zotero 7/8 plugin (TypeScript, esbuild) that mirrors Zotero's collection hierarchy as real filesystem directories, with per-paper folders containing PDF, Markdown, BibTeX, Kimi review, notes, and arXiv ID files.

Built on [zotero-plugin-template](https://github.com/windingwind/zotero-plugin-template) by windingwind.

## Architecture Overview

```
Zotero.Notifier events
    ↓
notifier.ts (dispatch + filter)
    ↓
exporter.ts (ExportQueue / incrementalRebuild / fullRebuild)
    ↓
tree-builder.ts (collection → directory mapping)
    ↓
content-providers/*.ts (pluggable per-file generators)
    ↓
filesystem (IOUtils / PathUtils)
```

### Data Flow

1. **Notifier** (`notifier.ts`) listens to `item`, `collection`, `collection-item` events
2. **Exporter** (`exporter.ts`) orchestrates: resolves paths via tree-builder, runs providers, manages index
3. **Tree Builder** (`tree-builder.ts`) maps Zotero collection hierarchy → filesystem directory tree
   - Each `CollectionNode` has `itemIds` (direct items) and `allItemIds` (direct + all descendant items)
   - **`Allin/` directory**: created when a collection has subcollections and `allItemIds.length > 0`. Contains **every paper from the collection and all its descendants** — a flat view of the entire subtree. This means even if the parent collection has no direct items, `Allin/` is still created as long as subcollections have papers.
4. **Content Providers** (`content-providers/*.ts`) generate individual files (PDF, Markdown, etc.)
5. **Index** (`.zofiles-index.json`) tracks exported items, paths, and files for incremental operations

## Key Files

| File                                         | Purpose                                                                   | When to touch                             |
| -------------------------------------------- | ------------------------------------------------------------------------- | ----------------------------------------- |
| `src/modules/exporter.ts`                    | Core export logic, queue, rebuild, index management                       | Export behavior, path resolution, cleanup |
| `src/modules/notifier.ts`                    | Event dispatch, progress notifications                                    | Adding new event handlers                 |
| `src/modules/tree-builder.ts`                | Collection → directory mapping, `Allin/` logic                            | Changing how collections map to folders   |
| `src/modules/preferences.ts`                 | Settings panel wiring, rebuild buttons                                    | Adding new preferences                    |
| `src/modules/content-providers/`             | Pluggable content generators                                              | Adding new export file types              |
| `src/modules/arxiv-id.ts`                    | arXiv ID extraction from multiple fields                                  | Changing paper identification             |
| `src/modules/utils.ts`                       | `ensureDir`, `joinPath`, `removeDir`, `formatPaperFolder`, `getCachePath` | Filesystem operations                     |
| `src/hooks.ts`                               | Plugin lifecycle (startup, shutdown, window load)                         | Init/cleanup logic                        |
| `addon/prefs.js`                             | Default preference values                                                 | Adding new settings                       |
| `addon/content/preferences.xhtml`            | Settings panel UI (XUL/XHTML)                                             | Settings UI changes                       |
| `addon/locale/{en-US,zh-CN}/preferences.ftl` | Localization strings                                                      | Any user-facing text                      |

## Preference Keys (addon/prefs.js)

| Key                  | Type          | Default                 | Description                                        |
| -------------------- | ------------- | ----------------------- | -------------------------------------------------- |
| `exportRoot`         | string        | `""`                    | Root directory for export tree                     |
| `cachePath`          | string        | `""`                    | Cache dir (falls back to `~/.cache/ZoFiles`)       |
| `paperFolderFormat`  | string        | `"{arxivId} - {title}"` | Paper folder naming template                       |
| `enabledCollections` | string (JSON) | `"[]"`                  | JSON array of enabled collection IDs (empty = all) |
| `exportPdf`          | bool          | `true`                  | Export PDF                                         |
| `exportMarkdown`     | bool          | `true`                  | Export full-text Markdown                          |
| `exportKimi`         | bool          | `true`                  | Export Kimi AI review                              |
| `exportBibtex`       | bool          | `true`                  | Export BibTeX                                      |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [X1AOX1A/ZoFiles](https://github.com/X1AOX1A/ZoFiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
