---
trigger: always_on
description: Design system reference for Urban Tech Creative website. Covers utilities, atoms, molecules, tokens, grid system, composition patterns, and design philosophy.
---


# UTC Design System

## Documentation Structure

- **`.cursor/rules/design-system.mdc`** (this file): Operational reference. Conventions, tokens, component APIs, composition patterns. Automatically available when editing `components/**` or `app/**`. This is the source of truth for "how to use things".
- **`docs/lld/*.md`**: Deep-dive implementation docs. Why things work the way they do, motion models, performance rationale, edge cases. The source of truth for "how things work internally". Read these when modifying a specific subsystem.

| Topic | Quick reference | Deep dive |
|-------|----------------|-----------|
| Design system | This file | -- |
| Cube (3D interaction) | Cube Face Mapping (below) | `docs/lld/cube.md` |
| Frame (semantic use) | Atoms section (below) | `docs/lld/frame.md` |
| UIGrid (layout grid) | Grid System section (below) | `docs/lld/ui-grid.md` |

---

## Atomic Design Hierarchy

### Utilities

- **Pressable** (`components/Pressable/Pressable.tsx`): Makes any subtree interactive and accessible. Zero visual opinion — no borders, backgrounds, padding, or hover effects. Renders `<Link>` for internal navigation (`href` starts with `/`), `<a target="_blank" rel="noopener noreferrer">` for external URLs (`href` starts with `http`), or `<button type="button">` for actions (no `href`). Props: `href?`, `onClick?`, `onMouseEnter?`, `onMouseLeave?`, `children`, `className?`, `aria-label?`, `aria-expanded?`, `aria-haspopup?` (for menu triggers), `data-testid?`. Use when wrapping custom compositions (Logo lockups, NavLinks) where the visual treatment is owned by the children.
- **NavMenu** (`components/Nav/NavMenu.tsx`): Handles show/hide and positioning of a dropdown menu. Renders a trigger (supplied by parent) and, when open, a panel positioned below it and right-aligned (`absolute top-full right-0`). Closes on click outside or Escape key. No visual opinion on panel content — parent provides trigger and panel (e.g. Frame + NavList). Props: `open`, `onClose`, `trigger` (ReactNode), `children` (panel content), `className?`, `panelClassName?`. Use for primary nav dropdown or any similar floating menu.
- **Icon**: Icons from `@phosphor-icons/react`. Prefer the **fill** variant for the acid design aesthetic — bold, solid shapes with strong silhouettes. Fall back to regular weight for secondary/supporting icons.

### Atoms (primitives)

- **Frame** (`components/Frame/Frame.tsx`): Post-neobrutalist bordered container. A self-contained piece of UI or content. Thick black borders (4px default). Selective rounded corners add friendliness — use sparingly (one curve is the sweet spot). Props: `borderSides` (omit sides to prevent double-borders when stacking adjacent Frames), `roundedCorners`, `borderWidth` (`border-2` | `border-4`). Purely presentational — no hover or interactive behaviour.
- **Accent** (`components/Accent/Accent.tsx`): Gradient bar placed alongside a Frame for visual emphasis. Draws the user's eye using colour. Gradients: `magenta-green`, `purple-orange`, `orange-purple`. Props: `direction` (`vertical` | `horizontal`), `gradient`, `borderSides` (match adjacent Frame to avoid double-borders). Purely presentational — no hover or interactive behaviour.
- **Heading** (`components/Heading.tsx`): Semantic heading component (h1-h6). Predefined size/weight classes per level. Uses Recursive font.
- **UIGrid** (`components/UIGrid/UIGrid.tsx`): Site-wide layout grid with square cells. Uses ResizeObserver to compute cell size so cells never stretch or squash. Use for LCARS-style panel positioning. Pair with **GridBlock** (`components/UIGrid/GridBlock.tsx`) for placement (col, row, colSpan, rowSpan). NOT for cube faces — use FaceGrid for those.
- **Overlay** (`components/Overlay/Overlay.tsx`): Full-viewport overlay with backdrop. Renders via portal (`document.body`). Props: `open` (boolean), `onClose` (backdrop click or Escape), `children`, `className`. CSS keyframe animations for fade + scale entrance. Handles focus management (auto-focus on mount, restore on close) and locks body scroll while open. Use as the base layer for any modal, detail panel, or dialog.

### Molecules (combinations of atoms)

- **Breadcrumbs** (`components/Breadcrumbs/Breadcrumbs.tsx`): Frame (theme-black bg, top-right curve, no border) + breadcrumb nav. Chevron separators, white text, cyan hover. Props: `items` (`{ label, path?, current? }[]`). Use for hierarchical page context (e.g. Work › Afghan Project).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [urban-tech-creative/utc-website](https://github.com/urban-tech-creative/utc-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
