---
trigger: always_on
description: Next.js 15 (App Router) · React 19 · Tailwind v4 · `motion` v12.
---

# Frontend Agent Guide (`apps/web`)

Next.js 15 (App Router) · React 19 · Tailwind v4 · `motion` v12.

The animated, high-contrast look is a deliberate product asset — a large part of
why this project gets attention. **Do not flatten it into a generic minimal
template.** The standing brief is: keep it distinctive, make it cheap. Every
rule below exists so those two can coexist.

## Build constraints

**This app builds on webpack, not Turbopack.** `npm run dev` intentionally omits
`--turbopack`. On the Next 15 line `@next/mdx` hands plugin functions to
`@mdx-js/loader`, and Turbopack serialises loader options across a process
boundary — the function arrives as `null` and MDX compilation dies with *"Cannot
use 'in' operator to search for 'plugins' in null"*. The string plugin form
Turbopack accepts is not resolved by this loader version, and
`experimental.mdxRs` cannot take arbitrary remark/rehype plugins. See the
comment at the top of `next.config.ts`. Revisit on Next 16.

Before proposing a Next 16 upgrade: `fumadocs-ui@16` and several other packages
hard-pin `next@16` / `react@^19.2`. It is a coordinated bump, not a one-liner.

## Design tokens — the one hard rule

`app/globals.css` is the single source of truth for colour, radius, motion and
elevation. **Use semantic tokens. Never reach for a raw Tailwind palette shade
in component code.**

```tsx
// Wrong — this is how the codebase ended up with five neutral ramps
<div className="bg-zinc-900 border-neutral-800 text-gray-400">
<span className="text-emerald-400">

// Right
<div className="bg-surface border-edge text-ink-muted">
<span className="text-brand">
```

| Purpose | Token |
| --- | --- |
| Page background | `bg-canvas` |
| Card / panel | `bg-surface` |
| Hover / inset | `bg-raised` |
| Popover, dialog | `bg-overlay` |
| Primary text | `text-ink` |
| Secondary text | `text-ink-muted` |
| Tertiary text | `text-ink-subtle` |
| Hairline | `border-edge` |
| Stronger line | `border-edge-strong` |
| Brand accent | `text-brand`, `bg-brand`, ramp `brand-50`…`brand-950` |
| On the accent | `text-brand-fg` |
| Status | `success`, `warning`, `danger`, `info` |
| Terminal chrome | `term-bg`, `term-chrome`, `term-edge`, `term-ink`, `term-muted`, `term-accent` |

### Graphite + Signal: the accent is rationed

The palette is near-monochrome — surfaces are true neutral at **zero chroma** —
with a single **amber** accent. That only works if the accent stays scarce.

**Amber means one thing: *this is the thing you are on*.** The primary action,
the live state, the selected row, the cursor. Aim for roughly **1–2% of the
pixels on screen**. If you are reaching for `text-brand` a third time on one
screen, the answer is `text-ink` or `text-ink-muted`.

Things that are explicitly *not* the accent's job:

- **Decoration.** No amber borders on every card, no amber icon on every list
  item, no gradient-filled headings.
- **Status.** `success`, `warning`, `danger`, `info` exist for that. Note
  `warning` sits at a yellower hue than the brand on purpose — amber-on-amber
  would make "provisioning" indistinguishable from "primary action".
- **Terminal output.** The 16 ANSI colours and `term-accent` follow shell
  convention. Green means passed, red means failed. Do not rebrand them.

Colour that is *not* the accent belongs in the backdrop. The landing hero's
CRT (`components/landing/hero/crt-backdrop.tsx`), the login wave panel
(`components/Auth/LoginShell.tsx`) and `AppBackdrop` carry the expressiveness
so the chrome can stay quiet.

Two naming traps:

- **`brand` is the amber accent. `accent` is not.** `accent` keeps its shadcn
  meaning — a subtle raised background for menu and dropdown hover — because a
  lot of vendored Radix code depends on it. Mapping `accent` to the brand colour
  turns every dropdown row bright amber.
- Prefer the existing utilities over re-deriving an effect inline: `glass`,
  `glow-brand`, `surface-card`, and `label` for the uppercase-mono UI voice.
  **Do not change `glass`** — it is the footer's original treatment, which
  the maintainer asked to keep as-is.

### Colour outside CSS

Canvas 2D, WebGL shaders, Satori (`next/og`) and the web manifest all parse
colour by hand and **cannot resolve `var()` or `oklch()`**. Passing a token to
`strokeStyle` silently paints black; passing one to a shader silently paints
its fallback. Import the hex mirrors from **`lib/tokens.ts`** instead, and if
you change a `--ds-*` value in globals.css, regenerate the matching entry
there — nothing enforces it at build time.

## Typography

Three families, declared in `app/fonts.ts`. Do not add a fourth.

| Role | Face | Utility |
| --- | --- | --- |
| Display — headlines, eyebrows | Space Grotesk | `font-display` |
| Body and UI | Geist | `font-sans` (default) |
| Code, terminal, paths, identifiers | Commit Mono | `font-mono` |

- **`font-display` means a display face, not a second mono.** It used to point
  at JetBrains Mono, and fifteen `<kbd>` and terminal call sites were relying on
  that. They now use `font-mono`, which is what they meant.
- Space Grotesk is display-only: x-height 0.486em, so it thins out below ~20px,
  and it has **no italic** — `font-style: italic` synthesises a slant.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ParthKapoor-dev/devex](https://github.com/ParthKapoor-dev/devex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
