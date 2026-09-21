---
trigger: always_on
description: You are building a web app for Meta Ray-Ban Display glasses using the `mrbd-ui-kit` component library. Follow these instructions precisely.
---

# AGENTS.md — AI Coding Assistant Instructions for mrbd-ui-kit

You are building a web app for Meta Ray-Ban Display glasses using the `mrbd-ui-kit` component library. Follow these instructions precisely.

## What is Meta Ray-Ban Display?

Meta Ray-Ban Display glasses have a 600×600 pixel monocular additive display projected into the right lens. Web apps are standard HTML/CSS/JS rendered on this display. Input comes from the Meta Neural Band (EMG wrist gestures) and a capacitive touch strip on the glasses temple, both mapped to arrow keys and Enter.

## Critical Display Rules

These are NOT suggestions. Violating them creates a broken experience on the hardware.

### DO:

- Use `<DisplayRoot>` as the outermost wrapper for all MRBD UI
- Use dark/transparent backgrounds (`bg-mrbd-accent/5`)
- Use `text-mrbd-text` for primary text (92% white, NOT pure white)
- Use `shadow-mrbd-glow` for emphasis effects (inner glow)
- Use `font-weight: 500` or higher for all text
- Use `Nunito` (or a fallback like `Noto Sans` for CJK/Thai/etc.) bold sans-serif font (weights 500+)
- Keep layouts right-anchored or F-pattern (display is in the right lens)
- Make all interactive elements spatially navigable via `<Focusable>` or composite components
- Give every `<Focusable>` and `<Button>` a unique `id`
- Keep content glanceable — users scan in <2 seconds
- Use `lucide-react` icons directly in your layouts, or pass them to components (like `<Button>`) that support them
- If needed, you can localize user-facing strings and customize the LoadingSpinner `label` for non-English screen readers

### NEVER:

- Use `#FFFFFF` or `rgb(255,255,255)` — causes ghosting. Use `text-mrbd-text` instead
- Use `drop-shadow` or `box-shadow` for decorative shadows — looks like dirt on lens. Use `shadow-mrbd-glow`
- Use `font-weight` below 500 — illegible on additive display
- Use mouse/touch event handlers as primary interaction — use spatial input events
- Create scrollable content without focus management — spatial input can't scroll
- Use heavy animations or frequent re-renders — battery-constrained device

## Project Setup

### Installation

```bash
npm install mrbd-ui-kit
```

### CSS Setup (global.css)

```css
@import "tailwindcss";
@import "mrbd-ui-kit/css";
```

This single import provides Tailwind v4 theme tokens (colors, shadows), focus ring styles, scrollbar hiding, transition defaults, and [`tailwindcss-text-box-trim`](https://www.npmjs.com/package/tailwindcss-text-box-trim) utilities (`box-trim-*`, `box-edge-*`) for pixel-perfect typographic spacing.

## Architecture

### Import Map

```typescript
// Components + client hooks (has "use client" banner)
import {
  Button,
  Card,
  DisplayRoot,
  Focusable,
  LoadingSpinner,
  Pill,
  ScrollArea,
  ScrollBar,
  ScrollContainer,
  Text,
} from "mrbd-ui-kit";

// Icons — use lucide-react directly
import { Check, Home, Search, Settings } from "lucide-react";

// Hooks
import { useSpatialInput, useFocusManager, usePreferredFocus, useIsMrbd, useScroll } from "mrbd-ui-kit";

// Server-side device detection (no "use client")
import { isMrbd, isMrbdFromHeaders } from "mrbd-ui-kit/server";

// Next.js RSC device detection (uses next/headers)
import { isMrbdServer } from "mrbd-ui-kit/next";
```

### Component Hierarchy

Every MRBD app must follow this structure:

```tsx
<DisplayRoot>
  {/* Your content */}
</DisplayRoot>
```

## Component Reference

### DisplayRoot

Root wrapper. Required. Sets up the 600×600 viewport, focus engine context, and keyboard event handling.

```tsx
<DisplayRoot
  focusOptions={{ wrap: true }}
  onSelect={(focusedId) => handleAction(focusedId)}
>
  {children}
</DisplayRoot>
```

Props: `children`, `className?`, `focusOptions?: { wrap?: boolean }`, `onSelect?: (id: string) => void`

### Text

Display-optimized typography. Enforces minimum font weight. Applies `box-trim-both box-edge-cap` by default to eliminate internal leading for pixel-perfect vertical alignment. Override with `box-trim-none` via `className` if needed.

```tsx
<Text size="lg" weight="bold">Title</Text>
<Text size="sm" className="text-gray-400">Subtitle</Text>
```

Props: `children`, `size?: 'sm' | 'md' | 'lg'`, `weight?: 'medium' | 'semibold' | 'bold'`, `as?: 'p' | 'span' | 'h1' | 'h2' | 'h3' | 'label'`, `dir?: 'ltr' | 'rtl' | 'auto'` (default `'auto'`), `className?`

### Focusable

Makes children spatially navigable. Every `id` must be unique. On Enter key press, fires `onSelect` and clicks the first child element.

```tsx
<Focusable id="item-1" onSelect={handleSelect} group="list">
  <div>content</div>
</Focusable>
```

Props: `id: string` (required), `children`, `group?: string`, `autoFocus?: boolean` (default `true` — set to `false` to skip initial auto-focus while keeping the element navigable), `onSelect?: () => void`, `onFocus?: () => void`, `onBlur?: () => void`, `disabled?: boolean`, `className?`


### Button

Spatially navigable button with variants. Default variant is `secondary`. Wraps `<Focusable>` internally. Applies `box-trim-both box-edge-cap` for precise text centering within the fixed button heights.

```tsx
import { Check, X } from "lucide-react";


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michaelcummingsofficial/mrbd-ui-kit](https://github.com/michaelcummingsofficial/mrbd-ui-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
