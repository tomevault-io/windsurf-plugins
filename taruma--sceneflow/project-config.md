---
trigger: always_on
description: This document provides instructions for AI agents and developers who need to maintain, extend, or modify the SceneFlow codebase.
---

# Agent Guidelines

This document provides instructions for AI agents and developers who need to maintain, extend, or modify the SceneFlow codebase.

## 1. Extending the Script Parser & Line Types

If you need to add a new script line type (e.g., `lyrics`, `transition`, or a specialized directive):

1. **Update Line Types**: Add the new type identifier to the `LineType` union in `src/lib/scriptProcessor.ts`:
   ```typescript
   export type LineType = 
     | 'name' 
     | 'speech' 
     | 'parenthetical' 
     | 'heading' 
     | 'note' 
     | 'effect' 
     | 'separator' 
     | 'part-separator' 
     | 'roman-title' 
     | 'action' 
     | 'default'
     | 'new-type';
   ```
2. **Implement Detection Logic**: Update `processScript()` in `src/lib/scriptProcessor.ts` with regex or heuristic rules to classify the line into your new type.
3. **Map Visual Styles**: Update `getLineClass()` in `src/lib/scriptStyles.ts` to return the appropriate Tailwind CSS classes, referencing active theme tokens (e.g., `theme.textColor`, `theme.textMutedColor`).

## 2. Modifying Styles, UI Tokens, & Script Themes

**DO NOT** write hardcoded Tailwind color classes directly into `src/App.tsx` or components for screenplay text, cue highlights, or modal containers.

- **Modular Design Tokens (`src/styles/tokens/`)**:
  - `ui.ts`: Centralized `UI_TOKENS` for layout shells (`layout`), modals & overlays (`modal`), dropdown menus (`dropdown`), buttons & action pills (`button`, including `supportPill` for creator tips and `xPill` for social updates), form controls (`input`), badges & time tags (`badge`, including `counter`, `timeCompact`, and `shortcut`), panel cards (`panel`), swatches (`swatch`), and alert containers (`alert`).
  - `src/index.css`: Semantic CSS custom properties defined in `:root` (`--app-bg`, `--surface`, `--border-main`, `--text-main`, `--overlay-bg`, `--color-support`) and mapped into Tailwind CSS v4's `@theme` directive.
  - `themes.ts`: Six visual themes configured in `SCRIPT_THEMES` (`light`, `warm`, `dark`).
  - `cues.ts`: Theme-calibrated RGB strings (`lightRgb`, `warmRgb`, `darkRgb`) defined across two curated palette profiles: `CUE_COLOR_DEFINITIONS_STANDARD` (360° balanced cinema spectrum) and `CUE_COLOR_DEFINITIONS_PROTANOPIA` (Red-Green Color Vision Deficiency safe mode with Deep Wine Shot). Resolved via `getCueColorForTheme(typeOrClass, themeId, paletteProfile)` with fallback normalization in `LEGACY_CLASS_MAP`.
  - `typography.ts`: Theme-specific structural classes and typography generated dynamically via `getScriptThemeStyles(themeId)`.
  - `helpers.ts`: Color manipulation and dynamic badge style generators (`hexToRgba`, `createCueBadgeStyle`, `createInlineCueStyle`).
- **Hook Integration (`useScriptTheme`)**: Use the `useScriptTheme(scriptThemeId, cuePaletteProfile)` hook in components to access active `themeStyles`, `themeMetadata`, `isDark`, and `resolveCueColor` helpers dynamically synchronized with the active accessibility profile.
- **Dynamic Category Indicator Invariant**: Category dot indicators across playback headers, dropdowns, and configuration modals (`ScriptHeaderControls`, `TimingSettingsModal`, `HighlightFilterBar`, `SyncCuesPanel`) must never use static Tailwind classes (`color.class`). They must resolve dynamically via `getCueColorForTheme(type, scriptThemeId, cuePaletteProfile)` to ensure accurate theme and CVD-safe palette rendering without contrast loss on active selection surfaces.
- **Dropdown Viewport Alignment Invariant**: Floating menus must anchor dynamically relative to viewport boundaries to eliminate offscreen clipping: menus on compact or left-aligned toolbars (mobile `ScriptHeaderControls`, `FileMenuDropdown`) must use left-anchoring (`left-0`, e.g. `UI_TOKENS.dropdown.menuLeft`); menus positioned at the far right on desktop (desktop `ScriptHeaderControls`, `SettingsMenuDropdown`) must anchor to the right (`lg:right-0 lg:left-auto` or `right-0`, e.g. `UI_TOKENS.dropdown.menuRight`) so they drop down cleanly into the reading canvas rather than overflowing past the right window frame.
- **Screenplay Cue Nomenclature & Typography Invariant**: Category and cue selectors (such as the Auto-Scroll "Focus Mode" dropdown) must render category names in uppercase with letter tracking (`uppercase tracking-wider`) to match standard screenplay industry formatting conventions (ALL CAPS sluglines and cues) and prevent title-casing acronym artifacts (e.g., ensuring VFX never renders as "Vfx").
- **Theming & Video Overlay Invariants (`.agents/rules/theming-and-overlay-invariants.md`)**: Strictly maintain two-tier independence between the App Shell (`themeMode` $\to$ `effectiveCategory`) and the Script Paper (`scriptThemeId` $\to$ `activeTheme.category`). When `pureBlackMode` is active on dark themes, DOM attributes (`data-pure-black-script` and `data-pure-black-shell`) ensure `#000000` backgrounds, stripped drop shadows, and hidden punch holes, while preserving `activeTheme.paperBorder`. Light and warm themes must remain completely untouched.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taruma/SceneFlow](https://github.com/taruma/SceneFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
