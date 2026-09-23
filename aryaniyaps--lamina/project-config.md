---
trigger: always_on
description: This guide explains the semantic design system philosophy for building consistent, maintainable UIs. The system is built on three core concepts: **Canvas**, **Surface**, and **Layer**.
---

# Design System Philosophy Guide

## Overview

This guide explains the semantic design system philosophy for building consistent, maintainable UIs. The system is built on three core concepts: **Canvas**, **Surface**, and **Layer**.

## Core Concepts

### 1. Canvas (`bg-canvas`)

**What it is**: The application-level background that serves as the foundation for all content. The canvas is the **entire application background**, not individual pages. There is only **one canvas** in the entire application, used at the root level.

**When to use**:

- **Only at the application root** - the single root container that wraps the entire application
- The main application background (not page backgrounds)

**When NOT to use**:

- ❌ Page-level backgrounds
- ❌ Nested containers
- ❌ Cards or components
- ❌ Modals or dropdowns
- ❌ Sidebars or panels
- ❌ Anywhere else in the application

**Critical Rule**: Canvas should only appear **once** in your entire application - at the root level. All pages, routes, and components sit on top of this single canvas.

**Example**:

```tsx
// ✅ Correct: Canvas at application root (only place it should be)
// App.tsx or root layout
<div className="bg-canvas min-h-screen">
  {/* All application content goes here */}
  <Routes>
    <Route path="/" element={<Page />} />
  </Routes>
</div>;

// ✅ Correct: Pages use surfaces, not canvas
function Page() {
  return <div className="bg-surface-1">{/* Page content */}</div>;
}

// ❌ Wrong: Canvas used for a page
function Page() {
  return <div className="bg-canvas">{/* Don't use canvas here */}</div>;
}

// ❌ Wrong: Canvas used for a card
<div className="bg-canvas p-4 rounded-md">{/* Card content */}</div>;
```

### 2. Surface (`bg-surface-1`, `bg-surface-2`, `bg-surface-3`)

**What it is**: Top-level containers that sit directly on the canvas. Surfaces never overlap each other - they are siblings in the layout hierarchy.

**When to use**:

- Main content areas
- Sections of a page
- Primary containers
- Panels that sit side-by-side

**Surface hierarchy**:

- `bg-surface-1`: Primary surface (most common)
- `bg-surface-2`: Secondary surface (for variation)
- `bg-surface-3`: Tertiary surface (rare, for special cases)

**Rules**:

- Surfaces are **siblings**, not nested (in the same plane)
- Each surface should use its corresponding layer for nested elements
- Surfaces provide the base for stacking layers

**Exception - Different Planes**:

- Modals, overlays, and popovers exist on a **different plane** (different z-index/stacking context)
- In these cases, it's acceptable to use a surface even when there's a surface below
- This is because they are visually and functionally separate from the underlying content

**Example**:

```tsx
// ✅ Correct: Surfaces as siblings
<div className="bg-canvas">
  <div className="bg-surface-1">
    {/* Main content area */}
  </div>
  <div className="bg-surface-2">
    {/* Secondary content area - sibling, not nested */}
  </div>
</div>

// ✅ Correct: Page with header and main (same surface)
<div className="bg-surface-1">
  <header className="border-b border-subtle">
    {/* Header is part of the surface, not a separate surface */}
  </header>
  <main>
    {/* Main is part of the surface, not a separate surface */}
  </main>
</div>

// ❌ Wrong: Surface nested in surface (same plane)
<div className="bg-surface-1">
  <div className="bg-surface-2">
    {/* This breaks the philosophy */}
  </div>
</div>

// ✅ Correct: Modal on different plane
<div className="bg-canvas">
  {/* Main page content */}
  <div className="bg-surface-1">
    Page content
  </div>

  {/* Modal overlay - different plane */}
  <div className="fixed inset-0 z-50">
    <div className="bg-backdrop fixed inset-0" />
    <div className="bg-surface-1 rounded-lg shadow-lg p-6">
      {/* Modal can use surface-1 even though page uses surface-1 */}
      Modal content
    </div>
  </div>
</div>
```

### 3. Layer (`bg-layer-1`, `bg-layer-2`, `bg-layer-3`)

**What it is**: Stacking layers that create depth within a surface. Layers stack on top of each other in a specific order.

**When to use**:

- Cards within a surface
- Group headers
- Nested containers
- Dropdowns and modals
- Sidebars
- Any element that needs to appear "on top" of a surface

**Layer hierarchy**:

- `bg-layer-1`: First layer (closest to surface)
- `bg-layer-2`: Second layer (on top of layer-1)
- `bg-layer-3`: Third layer (on top of layer-2)

**Critical Rule - Layer-to-Surface Association**:

- `bg-surface-1` → use `bg-layer-1` for nested elements
- `bg-surface-2` → use `bg-layer-2` for nested elements
- `bg-surface-3` → use `bg-layer-3` for nested elements

**Rare Exception - Visual Separation**:

In very rare cases, you may go one level above for visual separation when needed for specific UI elements:

- Inputs in modals (modal has `bg-surface-1`, input can use `bg-layer-2` for separation)
- Buttons, switches, and form controls that need more visual distinction
- **Important**: This is very rare and should only be used for interactive form elements, not for content boxes or cards

**Example**:

```tsx
// ✅ Correct: Surface-1 with layer-1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aryaniyaps/lamina](https://github.com/aryaniyaps/lamina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
