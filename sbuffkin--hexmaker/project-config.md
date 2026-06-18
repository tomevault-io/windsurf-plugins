---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development (watch mode with inline sourcemaps) — use with Hot Reload (see below)
npm run dev

# Production build (type-checks then bundles)
npm run build

# Run tests (vitest)
npm test

# Bump version (updates manifest.json and versions.json, stages both)
npm run version
```

Slash commands (invoke inside Claude Code):
- `/dev` — starts esbuild in watch mode (long-running; pairs with Hot Reload)
- `/rebuild` — one-off production build + test run; reports errors if either fails

**Always run `/rebuild` after making code changes.** The TypeScript type-check and test suite are the only automated verification — catch errors before the user does.

## Dev loop

Each coding session:
1. Run `npm run dev` in a terminal (or `/dev` from Claude Code) — esbuild watches for changes and rebuilds `main.js` on every save
2. After a rebuild, reload the plugin in Obsidian:
   ```bash
   powershell.exe -Command "obsidian plugin:reload id=duckmage-plugin"
   ```
3. Use `/rebuild` for a final production build before committing (runs the TypeScript type-check and tests too)

## Architecture

The plugin source is split across `main.ts` (entry point re-export) and modules under `src/`. esbuild bundles everything into `main.js`, which Obsidian loads directly.

See `ARCHITECTURE.md` for a full overview of the system.

### Source layout

```
main.ts                          ← thin re-export: export { default } from "./src/DuckmagePlugin"
src/
  DuckmagePlugin.ts              ← Plugin class (entry point, default export)
  HexMapView.ts                  ← ItemView — hex grid, all drawing tools, inline modals
  HexEditorModal.ts              ← Modal — right-click per-hex editor (terrain, links, notes, icon override)
  HexTableView.ts                ← ItemView — spreadsheet view of all hex notes with filters/sort
  TerrainPickerModal.ts          ← Modal — full terrain palette picker for the terrain paint tool
  TerrainEntryEditorModal.ts     ← Modal — edit a single terrain palette entry (name, color, icon)
  IconPickerModal.ts             ← Modal — icon picker for the icon paint tool
  RegionModal.ts                 ← Modal — switch active region, create/rename/delete regions
  FileLinkSuggestModal.ts        ← SuggestModal — file picker scoped to worldFolder
  RandomTableView.ts             ← ItemView — random table + workflow browser (tabbed: Tables / Workflows)
  RandomTableModal.ts            ← Modal — inline roll modal used from HexEditorModal
  RandomTableEditorModal.ts      ← Modal — edit entries of a random table file
  WorkflowEditorModal.ts         ← Modal — edit workflow definition (steps, template, results folder)
  WorkflowWizardModal.ts         ← Modal — execute a workflow (roll steps, fill template, save as note)
  randomTable.ts                 ← Pure logic: parse, roll, weight, die-range helpers
  workflow.ts                    ← Pure logic: parse/serialize workflows, generate templates, placeholder helpers
  DuckmageSettingTab.ts          ← PluginSettingTab — settings UI
  types.ts                       ← Interfaces & type constants (TerrainColor, DuckmagePluginSettings, LINK_SECTIONS, TEXT_SECTIONS)
  constants.ts                   ← Runtime constants (VIEW_TYPE_*, DEFAULT_TERRAIN_PALETTE, DEFAULT_SETTINGS)
  defaultHexTemplate.md          ← Built-in hex note template (imported as text via esbuild loader)
  frontmatter.ts                 ← YAML frontmatter helpers (terrain + icon override read/write)
  sections.ts                    ← Markdown section helpers (addLinkToSection, getLinksInSection, getAllSectionData, …)
  utils.ts                       ← Shared utilities (normalizeFolder, getIconUrl, makeTableTemplate)
  md.d.ts                        ← TypeScript declaration for "*.md" text imports
```

The `.md` loader is configured in `esbuild.config.mjs` (`loader: { '.md': 'text' }`), allowing `defaultHexTemplate.md` to be imported as a plain string.

### Plugin purpose

Renders an interactive hex-grid map for tabletop RPG world-building inside Obsidian. Each hex cell corresponds to a Markdown note on disk. The map supports terrain painting, icon painting, road/river chain drawing, link-to-hex tools (random tables, factions), panning, and zooming. A spreadsheet view summarises all hex notes with filtering. A random tables view lets users browse, roll, and edit weighted random tables, and execute multi-step workflows that chain table rolls into a filled template note.

### Key classes

- **`DuckmagePlugin`** (`src/DuckmagePlugin.ts`) — Main plugin entry point. Registers views (`HexMapView`, `HexTableView`, `RandomTableView`), ribbon icons, commands, and the settings tab. Key public API:
  - `hexPath(x, y)` — vault-relative path for a hex note
  - `createHexNote(x, y)` — creates hex note from template
  - `loadAvailableIcons()` — merges plugin `icons/` with user custom icons folder into `availableIcons: string[]`
  - `refreshHexMap()` — re-renders all open `HexMapView` instances

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sbuffkin/hexmaker](https://github.com/sbuffkin/hexmaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
