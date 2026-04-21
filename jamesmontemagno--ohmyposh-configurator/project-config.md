---
trigger: always_on
description: Visual configurator for Oh My Posh terminal prompts. React 19 + TypeScript + Vite app with drag-and-drop interface. Runs 100% client-side, hosted on GitHub Pages at `/ohmyposh-configurator/`.
---

# Copilot Instructions for Oh My Posh Configurator

## Project Overview

Visual configurator for Oh My Posh terminal prompts. React 19 + TypeScript + Vite app with drag-and-drop interface. Runs 100% client-side, hosted on GitHub Pages at `/ohmyposh-configurator/`.

## Architecture

### Core Data Flow
```
SegmentPicker → Canvas → PropertiesPanel
       ↓           ↓            ↓
    (drag)     (blocks)    (edit segment)
              ↘    ↓    ↙
           configStore (Zustand)
                  ↓
             PreviewPanel
```

### Key Patterns

**State Management**: Two Zustand stores with persistence:
- [src/store/configStore.ts](src/store/configStore.ts) - Main config state with blocks, segments, tooltips, and global settings. Use `generateId()` for new blocks/segments/tooltips.
- [src/store/advancedFeaturesStore.ts](src/store/advancedFeaturesStore.ts) - Advanced feature toggles for progressive disclosure. Controls visibility of advanced UI controls across the app.

**Segment Metadata**: Stored as JSON in [public/segments/](public/segments/) by category (system, scm, languages, cloud, cli, web, music, health). Each segment defines `type`, `name`, `icon`, `defaultTemplate`, `properties`, `options`, and optionally `defaultCache`. Loaded dynamically via [src/utils/segmentLoader.ts](src/utils/segmentLoader.ts).

**Default Cache Settings**: Segment metadata includes optional `defaultCache` field with `duration` and `strategy`. When adding segments, these defaults are auto-applied. See Cache Strategy Guide in `public/segments/README.md`.

**Color Schemes**: Default colors in [src/data/colorSchemes.ts](src/data/colorSchemes.ts) - category-based with per-segment overrides. New segments inherit from category unless specified in `segmentColorOverrides`.

**Export/Import**: [src/utils/configExporter.ts](src/utils/configExporter.ts) strips internal `id` fields before export. Supports JSON/YAML/TOML. Unicode chars escaped as `\uXXXX` in JSON. Import automatically detects and enables advanced features when `autoDetectOnImport` is enabled.

## Development Commands

```bash
npm run dev          # Start Vite dev server
npm run build        # TypeScript check + Vite build
npm run validate     # Validate configs in public/configs/
npm run lint         # ESLint check
npm run test         # Run unit tests (Vitest)
```

## Code Conventions

### Components
- Located in [src/components/](src/components/), each major component has its own folder with `index.ts` re-export
- Use `NerdIcon` component for all icons (uses custom Nerd Font icons with unique IDs)
- **Icon Validation**: When using `NerdIcon`, always verify the icon ID exists in [src/constants/nerdFontIcons.ts](src/constants/nerdFontIcons.ts). Search the file first to find an appropriate existing icon, or add a new one if needed. Invalid icon IDs will render as "?" in the UI.
- Tailwind CSS with project colors: `#0f0f23` (bg-dark), `#1a1a2e` (bg-panel), `#0f3460` (borders), `#e94560` (accent)

### Types
- All Oh My Posh types in [src/types/ohmyposh.ts](src/types/ohmyposh.ts)
- `Segment` and `Block` have internal `id` fields (stripped on export)
- `SegmentMetadata` defines picker display info + available `properties` and `options`

### Adding New Segments
1. Add to appropriate JSON file in `public/segments/` (keep alphabetized)
2. Include `type`, `name`, `description`, `icon`, `defaultTemplate`
3. Add `properties` array for template variables (e.g., `.Full`, `.Path`)
4. Add `options` array for configurable settings with `name`, `type`, `default`, `description`
5. Optionally add `defaultCache` with `duration` and `strategy` (see Cache Strategy Guide in `public/segments/README.md`)
6. Optionally add color override in `colorSchemes.ts`

### Config Structure
- Sample configs: `public/configs/samples/` 
- Community configs: `public/configs/community/`
- Each category has `manifest.json` listing configs with metadata
- Config files contain ONLY Oh My Posh config (no wrapper metadata)

## Important Patterns

### Segment Options vs Properties
- **Properties**: Template variables like `{{ .Full }}` - display in template help
- **Options**: Runtime config like `home_enabled`, `fetch_version` - editable in `options` object

### Advanced Features System
- Use `advancedFeaturesStore` to check if advanced UI controls should be shown
- Pattern: `const { caching } = useAdvancedFeaturesStore(); if (!caching) return null;`
- Always respect feature flags when rendering advanced controls (aliases, templates logic, responsive width, folder filters, cache, interactive, color templates, diamond symbols, block overflow, filler, tooltips, extra prompts, console title, shell integration, palette variants)
- Auto-detection on import: When importing configs, scan for advanced features and enable them if `autoDetectOnImport` is true

### Tooltips Feature
- Tooltips are command-triggered custom prompts (e.g., show git status when typing `git`)
- Stored separately from blocks in config, managed by `addTooltip`, `updateTooltip`, `removeTooltip` actions
- Each tooltip has `id`, `tips` (trigger commands), and extends `Segment` interface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamesmontemagno/ohmyposh-configurator](https://github.com/jamesmontemagno/ohmyposh-configurator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
