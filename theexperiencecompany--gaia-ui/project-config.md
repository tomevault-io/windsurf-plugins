---
trigger: always_on
description: Hey there! Welcome to the Gaia UI library. This guide will walk you through everything you need to know about adding new components, maintaining design consistency, and keeping things beautiful.
---

# AGENTS.md

Hey there! Welcome to the Gaia UI library. This guide will walk you through everything you need to know about adding new components, maintaining design consistency, and keeping things beautiful.

## Quick Start: Adding a New Component

When you add a new component, you'll need to touch a few places. Here's the checklist:

### 1. Create the Component File

Add your component to `registry/new-york/ui/your-component.tsx`. This is where the actual component code lives.

```
registry/
└── new-york/
    └── ui/
        └── your-component.tsx  ← Your new component
```

### 2. Register It

Open `registry.json` in the project root and add your component to the `items` array:

```json
{
  "name": "your-component",
  "type": "registry:ui",
  "title": "Your Component",
  "description": "A brief, compelling description of what this component does.",
  "dependencies": ["any-npm-packages"],
  "registryDependencies": ["icons", "other-gaia-components"],
  "files": [
    {
      "path": "registry/new-york/ui/your-component.tsx",
      "type": "registry:ui"
    }
  ]
}
```

### 3. Add Preview Components

Create preview examples in `components/previews/your-component/`:

```
components/
└── previews/
    └── your-component/
        ├── default.tsx          ← Basic usage example
        ├── with-variants.tsx    ← Different variants
        └── custom-example.tsx   ← Any other demos
```

### 4. Write the Documentation

Create a docs page at `content/docs/components/your-component.mdx`:

```mdx
---
title: Your Component
description: What it does and why someone would use it.
---

<ComponentPreview name="your-component/default" />

## Usage

Show how to use it.

## Installation

<Tabs defaultValue="automatic" className="mt-4">
  <!-- Add both automatic and manual installation options -->
</Tabs>

## Props

Document all the props in a table.
```

### 5. Update Navigation (if needed)

If you're adding a new category or the component needs special placement, update `lib/navigation.ts`.

---

## Design Philosophy

We follow Apple's Human Interface Guidelines and modern design principles. Here's what that means in practice.

### Flat Design, No Outlines

Avoid heavy borders and outlines. Use these techniques instead:

- **Subtle backgrounds** — Differentiate elements with slight background color changes
- **Elevation through shadows** — Use soft, layered shadows for depth
- **Negative space** — Let elements breathe; whitespace is your friend
- **Color fills** — Solid, muted colors with proper opacity

```tsx
// ❌ Don't do this
<div className="border border-gray-300 rounded">

// ✅ Do this instead
<div className="bg-muted/50 rounded-xl shadow-sm">
```

### Visual Hierarchy

Guide the eye naturally through your component:

1. **Size matters** — Important elements should be larger
2. **Weight for emphasis** — Use font-weight 500-600 (medium to semibold) for headings, 400 for body text
3. **Color for attention** — Primary actions get the accent color, secondary actions stay muted
4. **Spacing for grouping** — Related items stay close, separate concerns stay apart

### Spacing & Padding

Be generous but intentional with space:

- **Touch targets** — Minimum 44x44px for interactive elements (Apple's recommendation)
- **Padding scale** — Stick to the Tailwind scale: `p-3`, `p-4`, `p-6`, `p-8`
- **Consistent gaps** — Use `gap-3` or `gap-4` for most layouts, `gap-6` or `gap-8` for section separation
- **Breathing room** — Cards need at least `p-4`, ideally `p-6`

```tsx
// Component padding guidelines
<Card className="p-6">                    // Outer container
  <CardHeader className="pb-4">           // Section spacing
    <CardTitle className="mb-2">          // Title to content
  </CardHeader>
  <CardContent className="space-y-4">     // Content gaps
```

### Typography

- **Headings** — Use the default font stack, weights 500-700
- **Body text** — 14-16px, weight 400, good line-height (1.5-1.6)
- **Labels** — 12-14px, weight 500, slightly muted color
- **Don't mix too many sizes** — Stick to 2-3 sizes per component max

### Color Usage

- **Primary/Accent** — Use sparingly for CTAs and important actions
- **Muted tones** — Background fills, secondary text, dividers
- **Semantic colors** — Red for destructive, green for success, yellow for warnings
- **Dark mode support** — Always design for both modes; use CSS variables

---

## Accessibility (A11y) – Non-Negotiable

Every component must be accessible. No exceptions.

### Keyboard Navigation

- All interactive elements must be focusable
- Logical tab order (follows visual flow)
- Visible focus indicators (don't remove outlines without adding custom focus styles)
- Support `Enter` and `Space` for activation
- `Escape` should close modals/dropdowns

```tsx
// Use proper interactive elements
<button onClick={...}>Submit</button>     // ✅ Accessible
<div onClick={...}>Submit</div>           // ❌ Not accessible

// If you must use div, add roles and keyboard handling
<div 
  role="button"
  tabIndex={0}
  onClick={...}
  onKeyDown={(e) => e.key === 'Enter' && handleClick()}
>
```

### Screen Readers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theexperiencecompany/gaia-ui](https://github.com/theexperiencecompany/gaia-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
