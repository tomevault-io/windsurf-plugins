---
trigger: always_on
description: This is **AI Research Archive** — a Vite + React 19 + Tailwind v4 + TypeScript app, deployed on Vercel. It hosts a curated AI/ML curriculum, resource library, jobs feed, and community directory.
---

# CLAUDE.md — Project guide for Claude

This is **AI Research Archive** — a Vite + React 19 + Tailwind v4 + TypeScript app, deployed on Vercel. It hosts a curated AI/ML curriculum, resource library, jobs feed, and community directory.

## The design system is non-negotiable

The project ships under the **General Agents Design System** (canonical zip: `~/Downloads/General Agents Design System.zip`, mirrored visual reference in `/tmp/ga-ds/` during dev sessions). **Use it.** Don't introduce ad-hoc colors, shadows, fonts, or layout primitives.

### Where the system lives

| Area | Location |
|---|---|
| Tokens (colors, type, radii, shadows, spacing) | `src/index.css` — `:root { --ga-* }` block + `@theme inline` token bridge |
| Brand fonts (Overused Grotesk, IBM Plex Mono, Departure Mono) | `public/fonts/` + `@font-face` in `src/index.css` |
| Brand logos / wordmarks | `public/assets/brand/` + `public/logo.png` (the pink `research archive` mark, also the favicon) |
| Brand React primitives | `src/components/brand/` — `Logo`, `Sparkle`, `PhIcon`, `ZigDivider`, `Sidebar`, `TopBar` |
| App shell | `src/components/Layout.tsx` + `Header.tsx` + `Sidebar.tsx` (uses brand primitives) |
| shadcn/ui primitives, brand-skinned | `src/components/ui/` |
| Live design showcase | `/design` route — renders every primitive against the canonical preview HTMLs |

### Foundational rules

- **Light mode only.** Dark was intentionally dropped — keycap buttons, layered soft shadows, and warm off-white surfaces don't survive a token-only invert. Don't add a `.dark` block back without a deliberate redesign pass.
- **Page background is `#FBFBFB` (`--ga-bg`)**, never pure white. White (`--ga-surface`) is reserved for cards, inputs, popovers — surfaces that need to lift.
- **Cards** = white surface + 12px radius + soft layered shadow + **NO border**. Use the `.grid-card` class. Hover lift only fires when the card is `<a>` / `<button>` / `data-interactive="true"`. Static info cards stay flat.
- **Primary buttons** = the GA "keycap" — black gradient `linear(180°, #272727 → #414141)` with `inset 0 1px 1px rgba(255,255,255,0.48)` and `inset 0 -3px 0 #232323`. Use `<Button>` from `src/components/ui/button.tsx` (which wraps `.ga-btn-primary`). Never raw `<button class="bg-black">`.
- **Sentence case for headings.** ALL-CAPS only for monospace meta labels via `.ga-mono-label` (IBM Plex Mono, 11–12px, letter-spacing 0.08em).
- **No emoji. No exclamation marks.** The product is calm.
- **Iconography = Phosphor Icons** loaded via CDN (regular / fill / bold weights). Use `<PhIcon name="..." size={N} />` from `@/components/brand`. Never import from `lucide-react` for new code — the existing usages are residual.
- **Section breaks use `<ZigDivider label="…" />`** (the wavy hand-drawn line). Never a flat `<div className="h-px bg-border" />`.
- **Tabs / filter pills** use the `.ga-tab` class with `data-active={bool}`. Never bordered chrome.
- **Colored category badges are forbidden.** Tags/badges are neutral mono chips: `inline-flex px-2 py-0.5 rounded bg-secondary text-muted-foreground text-[10px] uppercase tracking-wider font-mono`. The only exceptions are third-party brand identity (YouTube red, GitHub black, Discord blurple) when shown as their own logo.

### Token quick reference

| Token | Value | Use for |
|---|---|---|
| `--ga-bg` | `#FBFBFB` | Page background |
| `--ga-surface` | `#FFFFFF` | Cards, inputs, popovers |
| `--ga-sidebar` | `#F8F8F8` | Sidebar bg |
| `--ga-hover` | `#EFEFEF` | Hover bg, active sidebar row |
| `--ga-chip` | `#F3F3F3` | Resting chip bg |
| `--ga-divider` | `#E9E9E9` | Hairline divider |
| `--ga-border` | `#E1E1E1` | Default border |
| `--ga-border-strong` | `#CFCFCF` | Popover ring, emphasized border |
| `--ga-fg1` | `#272727` | Primary text |
| `--ga-fg2` | `#929292` | Secondary text |
| `--ga-fg3` | `#ADADAD` | Tertiary / placeholder |
| `--ga-ink-900` | `#1D1D1D` | Button dark |
| `--ga-sparkle-from / -to` | `#A78BFA → #6366F1` | Ronika sparkle gradient (only saturated brand color — accent only) |
| `--ga-font-sans` | Overused Grotesk | UI body text |
| `--ga-font-mono` | IBM Plex Mono | Mono labels, numerics |
| `--ga-font-departure` | Departure Mono | Pixel-grid (brand wordmark, hero treatments) |
| `--ga-r-sm / md / lg / xl` | `6 / 8 / 12 / 16` px | Inputs / buttons / cards / modals |
| `--ga-shadow-sm / md / lg` | layered soft | Resting / elevated / overlay surfaces |

shadcn semantic vars (`--background`, `--foreground`, `--primary`, etc.) all reference the `--ga-*` tokens above. So `bg-background`, `text-muted-foreground`, etc. flow through the canonical palette automatically.

Legacy `--color-bg`, `--color-text`, `--color-text-muted` etc. utilities are aliased too — they exist for backward-compat in a few residual files and resolve to the canonical brand. Don't introduce new usages; prefer `text-foreground`, `text-muted-foreground`, `bg-card`, etc.

### When asked to build something new

Reach for these in this order:

1. **Existing brand primitive** (`Logo`, `Sparkle`, `ZigDivider`, `Sidebar`, `TopBar`, `PhIcon`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suraj-xd/ai-research-archive](https://github.com/suraj-xd/ai-research-archive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
