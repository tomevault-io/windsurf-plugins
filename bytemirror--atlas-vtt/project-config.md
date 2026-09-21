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
- **Reuse shared components**: Toolbar buttons must use the `ToolButton` component. Don't create custom button markup/classes that duplicate what `ToolButton` + `Button` already provide.

# Memory Notes

## Successfully Hiding Obsidian Tabs for Note Preview
To hide tabs while maintaining full editing functionality in the note preview window:

1. **CSS Approach**: Add styles to hide tab headers with a specific data attribute:
   ```css
   .workspace-tab-header[data-atlas-preview="true"] {
     display: none !important;
   }
   .workspace-tab-container:has(.workspace-tab-header[data-atlas-preview="true"]):not(:has(.workspace-tab-header:not([data-atlas-preview="true"]))) {
     display: none !important;
   }
   ```

2. **Leaf Management**: 
   - Create a new leaf with `this.app.workspace.getLeaf(true)`
   - Mark the leaf and its tab header with `data-atlas-preview="true"` attribute
   - **Critical**: Call `leaf.detach()` to remove the leaf from the main workspace split
   - This prevents the leaf from affecting the active view while keeping it functional

3. **View Preservation**:
   - Store the original active leaf before creating the preview leaf
   - After opening the file in the preview leaf, restore the original active leaf
   - Use `this.app.workspace.setActiveLeaf(originalActiveLeaf, { focus: false })`

4. **Content Rendering**:
   - After detaching, manually append the leaf's view container to the preview window
   - This maintains full editing capabilities while preventing workspace navigation

This approach ensures the user stays on the map canvas while previewing notes with full editing support.

## Hexagonal Grids
Hex geometry lives in `src/app/grid/hexGeometry.ts` and rendering in `src/app/grid/hexGridDrawer.ts`.

- **Size convention**: `grid.size` is the flat-to-flat distance of a hex (width of a pointy-top hex, height of a flat-top hex), the same convention Foundry VTT and Owlbear Rodeo use. Circumradius is `size / sqrt(3)`. A size-1 token therefore has the same pixel diameter on hex and square grids.
- **Orientation**: `hex-vertical` = pointy-top (rows), `hex-horizontal` = flat-top (columns).
- **Coordinates**: axial `(q, r)` with cube rounding for pixel-to-hex (Red Blob Games). Never round `q` and `r` independently.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ByteMirror/atlas-vtt](https://github.com/ByteMirror/atlas-vtt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
