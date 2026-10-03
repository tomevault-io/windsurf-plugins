---
trigger: always_on
description: - We are working on Atlas VTT, a Virtual Tabletop plugin for Obsidian.md
---

# Context
- We are working on Atlas VTT, a Virtual Tabletop plugin for Obsidian.md
- **IMPORTANT**: We use PIXI.js v8 (not v7). Obsidian bundles PIXI v7 globally, but we must use our own PIXI v8 imports.

# Coding pattern preferences
- Adhere to the single responsibility principle SOC
- always prefer best practice solutions
- Avoid duplication of code whenever possible, which means checking for
other areas of the codebase that might already have similar code and
functionality
- Before you code anything with PixiJS or alter Pixi code always check online first if you are adhering to the newest pixiJS V8 best practices.
- If you believe that you would benefit from up to date information on some framework when e.g fixing a warning about PixiJS v8, please just use web search without asking.
- You are careful to only make changes that are requested or you are
confident are well understood and related to the change being requested
- When fixing an issue or bug, do not introduce a new pattern or
technology without first exhausting all options for the existing
implementation. And if you finally do this, make sure to remove the old
ipmlementation afterwards so we don't have duplicate logic.
- Keep the codebase very clean and organized
- Avoid writing scripts in files if possible, egpecially if the script
is likely only to be run once
- Avoid having files over 200-300 lines of code. Refactor at that point.
- Mocking data is only needed for actual test files, never mock data otherwise
- Never add stubbing or fake data patterns to code 
- In Typescript always type out return types in functions to make onboarding new team members easier and to provide clearer intentions 
- Prefer SCSS classes and Obsidian native styles (see obsidian-colors.md). Tailwind utilities exist in older components and are scoped under `.atlas-vtt-plugin`; do not add new Tailwind usage.

# UI Design Principles
- **Uniform padding**: Every container must use equal padding on all sides and the same value for gaps between child elements. A tooltip, modal, or toolbar with `padding: 8px` must also use `gap: 8px` between its children. Never use asymmetric padding (e.g. `6px 12px`) unless there is an explicit, justified reason.
- **Consistent stroke/border style**: All elevated surfaces (toolbars, tooltips, modals, popovers) share the same border treatment defined in the `atlas-elevated-surface` mixin (`styles/_mixins.scss`). Use it instead of ad-hoc border values.
- **Nested border radius**: The mathematical formula is `inner = outer - padding`, but this only matters when elements are flush against the container edge. For elements separated by padding (buttons inside panels), step down one level in the radius scale instead: containers use `$radius-xl` (12px), inner elements use `$radius-l` (8px). Both look clearly rounded and the padding gap prevents the eye from comparing curves directly.
- **Close buttons**: every panel closes with `CloseButton` (Obsidian modals opened by Atlas add `ATLAS_NATIVE_MODAL_CLASSES` to `modalEl`). The panel sets its corner with `atlas-panel-radius($radius-2xl)` and its header with `atlas-close-header`, so the button sits `$close-button-gap` from the border with a concentric corner. Concentric means outer = inner + gap, and only on the corners that face the panel's corners: a small element keeps the control radius on its other corners, or its edges curve away from the panel's and the gap stops being even. When the inner radius comes out near 0, the panel's radius is too small, so raise it; never shrink the inner one. Elements near such a panel's corners use `atlas-panel-inset-radius($inset)`.
- **One corner language**: every floating surface (dialogs, panels, the toolbar and undo/redo bar, scene tab bar, widgets, initiative trackers, dice tray, toolbar dropdowns) uses `atlas-panel-radius($radius-2xl)` with its corner controls $spacing-s inside the border; context menus are tighter: their rows are 28px capsules $spacing-xs inside the border, all rounded alike, and the menu's corner is derived from them so every row stays concentric (`atlas-context-menu.scss`); bars shorter than 48px render as capsules, so their end controls cap the inset radius at half their height (`atlas-panel-inset-radius-value($inset, $max)`). Surfaces and inset elements also take Obsidian's `corner-shape` (`atlas-corner-shape`, a superellipse on macOS): Obsidian gives every `<button>` that shape, and a squircle inside a round corner never keeps an even gap. Panels built with the DOM instead of React open and close with `animatePanelIn` / `animatePanelOutAndRemove` (`src/app/ui/panelMotion.ts`), the same motion as the dialogs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atlas-vtt/atlas-vtt](https://github.com/atlas-vtt/atlas-vtt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
