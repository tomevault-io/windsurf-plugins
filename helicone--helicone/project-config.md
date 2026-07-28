---
trigger: always_on
description: Helicone Style Guidelines
---

# Helicone Design System Guidelines

## Core Principles
1. Use semantic HTML elements
2. Use Tailwind utility classes for colors
3. Use typography components for consistent text styling
4. Use Tailwind's spacing scale
5. Use lucide-react for icons

## Typography System

### Usage Patterns
```tsx
// ✅ Do this
import { H1, P, Small, Muted } from "@/components/ui/typography"

<H1>Page Title</H1>
<P>Regular paragraph</P>
<Small>Helper text</Small>
<Muted>Secondary text</Muted>

// ❌ Don't do this
<h1 className="text-4xl font-bold">Raw Styles</h1>
<H1 className="text-3xl">Overriding Typography</H1>
<P className="text-sm">Wrong Size</P>
```

### Available Components
```tsx
// Headings
<H1>              // Main page titles
<H1Large>         // Hero sections
<H2>              // Section headers
<H3>              // Subsection headers
<H4>              // Card titles

// Body Text
<P>               // Regular paragraphs
<Lead>            // Introduction text
<Large>           // Emphasized body text
<Small>           // Helper text
<Muted>           // Secondary text

// Special Elements
<Blockquote>      // Quotations
<Code>            // Inline code
<List>            // Unordered lists

// Table Elements
<TableHead>       // Table headers
<TableCell>       // Table cells
```

### Typography Scale
| Component | Size | Line Height | Weight | Usage |
|-----|---|----|---|----|
| H1 | text-3xl (30px) / lg:text-4xl (36px) | leading-10 (40px) | semibold | Main page titles |
| H1Large | text-5xl (48px) | leading-10 (40px) | extrabold | Hero sections |
| H2 | text-3xl (30px) | leading-9 (36px) | semibold | Section headers |
| H3 | text-2xl (24px) | leading-8 (32px) | semibold | Subsection headers |
| H4 | text-xl (20px) | leading-7 (28px) | semibold | Card titles |
| Lead | text-xl (20px) | leading-7 (28px) | normal | Introduction text |
| P | text-base (16px) | leading-7 (28px) | normal | Body text |
| Small | text-sm (14px) | leading-4 (16px) | medium | Helper text |
| Muted | text-sm (14px) | leading-none | normal | Secondary text |

## Color System

### Usage Patterns
```tsx
// 1. Use Tailwind utility classes for colors
✅ text-foreground
✅ bg-background
✅ border-border

// 2. Use foreground variants for text on colored backgrounds
✅ bg-primary text-primary-foreground
✅ bg-destructive text-destructive-foreground

// ❌ Don't use CSS variables directly
❌ text-[hsl(var(--foreground))]
❌ bg-[hsl(var(--background))]

// ❌ Don't use raw colors
❌ text-slate-900
❌ bg-white
```

### Color Utility Classes
```tsx
/* Text Colors */
text-foreground              // Primary text
text-muted-foreground        // Secondary text
text-accent-foreground       // Accent text

/* Background Colors */
bg-background                // Page background
bg-card                      // Card background
bg-muted                     // Muted background

/* Border Colors */
border-border                // Default borders
border-input                 // Form inputs
ring-ring                    // Focus rings
```

## Color Reference Values

### Primary Colors
```tsx
/* Light Mode */
bg-primary                   // #0EA5E9 (sky-500)
text-primary-foreground      // #F0F9FF (sky-50)

/* Dark Mode */
bg-primary                   // #0284C7 (sky-600)
text-primary-foreground      // #083344 (sky-950)
```

### Background Colors
```tsx
/* Light Mode */
bg-background                // #FFFFFF (white)
text-foreground              // #0F172A (slate-900)

/* Dark Mode */
bg-background                // #020617 (slate-950)
text-foreground              // #F8FAFC (slate-50)
```

### Accent Colors
```tsx
/* Light Mode */
bg-accent                    // #F1F5F9 (slate-100)
text-accent-foreground       // #0F172A (slate-900)

/* Dark Mode */
bg-accent                    // #1E293B (slate-800)
text-accent-foreground       // #F8FAFC (slate-50)
```

### UI Component Colors
```tsx
/* Light Mode */
bg-card                      // #FFFFFF (white)
text-card-foreground         // #020617 (slate-950)
bg-popover                   // #FFFFFF (white)
text-popover-foreground      // #020617 (slate-950)
border-border                // #E2E8F0 (slate-200)
border-input                 // #E2E8F0 (slate-200)
ring-ring                    // #0F172A (slate-900)

/* Dark Mode */
bg-card                      // #020617 (slate-950)
text-card-foreground         // #F8FAFC (slate-50)
bg-popover                   // #020617 (slate-950)
text-popover-foreground      // #F8FAFC (slate-50)
border-border                // #1E293B (slate-800)
border-input                 // #1E293B (slate-800)
ring-ring                    // #CBD5E1 (slate-300)
```

### Semantic Colors
```tsx
/* Light Mode */
bg-muted                     // #F1F5F9 (slate-100)
text-muted-foreground        // #64748B (slate-500)
bg-secondary                 // #F1F5F9 (slate-100)
text-secondary-foreground    // #0F172A (slate-900)
bg-destructive               // #DC2626 (red-600)
text-destructive-foreground  // #450A0A (red-950)
bg-confirmative              // #16A34A (green-600)
text-confirmative-foreground // #052E16 (green-950)

/* Dark Mode */
bg-muted                     // #1E293B (slate-800)
text-muted-foreground        // #94A3B8 (slate-400)
bg-secondary                 // #1E293B (slate-800)
text-secondary-foreground    // #F8FAFC (slate-50)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Helicone/helicone](https://github.com/Helicone/helicone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
