---
trigger: always_on
description: # Design System & UI Guidelines
---


# bossROD TV Design System & UI Guidelines

## Core Aesthetic: "Minimalist Tech & Code"

The bossROD TV brand aesthetic sits at the intersection of **Programming, Technology, and Content Creation**. It is **"Clean Code meets Visual Clarity."**

The design must instantly communicate four key pillars:

1.  **Technical Excellence:** Clean, precise layouts that reflect code structure and logical thinking.
2.  **Minimalist Focus:** Black and white palette that removes distraction and emphasizes content.
3.  **Developer Identity:** Terminal aesthetics, monospace typography, and code-inspired UI patterns.
4.  **Content Creation:** Video frames, streaming UI elements, and creator-focused components.

**Key Visual Metaphors:**

- **The Terminal:** Dark backgrounds, monospace fonts, command-line aesthetics.
- **The IDE:** Syntax highlighting accents, tab-based navigation, panel layouts.
- **The Triangle:** Logo-inspired angular shapes, geometric precision, sharp edges.
- **The Stream:** Video frames, live indicators, recording UI elements.

---

## 1. Typography & Text Effects

### Font Stack

- **Primary:** `Inter` (via Tailwind default sans) for UI and body text.
- **Code/Accent:** `JetBrains Mono` or system monospace for technical elements and emphasis.

### Text Colors (Theme-Aware)

**CRITICAL:** ALWAYS use semantic theme colors (`text-foreground`, `text-muted-foreground`, `text-primary`, `bg-background`, `bg-card`). **NEVER** use hardcoded colors like `text-gray-900`, `text-gray-600`, or `bg-white` (unless intentionally fixed, e.g., on a dark overlay).

| Role           | Class                   | Description                              |
| -------------- | ----------------------- | ---------------------------------------- |
| **Headings**   | `text-foreground`       | Main titles, high contrast.              |
| **Body**       | `text-muted-foreground` | Descriptions and secondary text.         |
| **Code/Tech**  | `font-mono`             | Technical terms, code snippets.          |
| **Emphasis**   | `text-primary`          | Key brand moments or active states.      |

### The "Code Comment" Effect

Use this for section subtitles to emphasize the **developer/tech** identity. It simulates code comments.

```tsx
<p className="font-mono text-sm text-muted-foreground">
  <span className="text-muted-foreground/60">// </span>
  Building the future, one commit at a time
</p>
```

### The "Terminal Prompt" Effect

Use for CTAs or interactive elements to create a command-line feel.

```tsx
<span className="font-mono text-sm">
  <span className="text-primary">$</span> start learning
  <span className="animate-pulse">_</span>
</span>
```

---

## 2. Color System

### Primary Palette: Black & White

The core palette is strictly monochromatic for maximum clarity and focus.

| Token              | Light Mode          | Dark Mode           | Usage                    |
| ------------------ | ------------------- | ------------------- | ------------------------ |
| `--background`     | `#ffffff`           | `#0a0a0a`           | Page backgrounds         |
| `--foreground`     | `#0a0a0a`           | `#fafafa`           | Primary text             |
| `--card`           | `#ffffff`           | `#111111`           | Card backgrounds         |
| `--card-foreground`| `#0a0a0a`           | `#fafafa`           | Card text                |
| `--muted`          | `#f4f4f5`           | `#1a1a1a`           | Muted backgrounds        |
| `--muted-foreground`| `#71717a`          | `#a1a1aa`           | Secondary text           |
| `--border`         | `#e4e4e7`           | `#27272a`           | Borders and dividers     |
| `--primary`        | `#0a0a0a`           | `#fafafa`           | Primary actions          |
| `--primary-foreground`| `#fafafa`        | `#0a0a0a`           | Text on primary          |

### Accent Colors (Sparingly Used)

Only use accents for functional feedback, never for decoration.

| Color    | Usage                                    |
| -------- | ---------------------------------------- |
| `green`  | Success states, "live" indicators        |
| `red`    | Error states, recording indicators       |
| `yellow` | Warning states, pending actions          |
| `blue`   | Links, information (use minimally)       |

---

## 3. Component Patterns

### A. The "Code Block" Card

Used for: _Features, tutorials, technical content._ Represents the **Developer Identity** aspect.

- **Container:** Dark background with subtle border, monospace accents.
- **Header:** Simulates a terminal/IDE title bar with dots.

```tsx
<div className="overflow-hidden rounded-lg border border-border bg-card">
  {/* Terminal Header */}
  <div className="flex items-center gap-2 border-b border-border bg-muted px-4 py-2">
    <div className="h-3 w-3 rounded-full bg-red-500/80" />
    <div className="h-3 w-3 rounded-full bg-yellow-500/80" />
    <div className="h-3 w-3 rounded-full bg-green-500/80" />
    <span className="ml-2 font-mono text-xs text-muted-foreground">
      component.tsx
    </span>
  </div>
  {/* Content */}
  <div className="p-4">
    <pre className="font-mono text-sm text-foreground">
      {/* Code content */}
    </pre>
  </div>
</div>
```

### B. The "Panel" Card (Clean Container)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/constROD) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
