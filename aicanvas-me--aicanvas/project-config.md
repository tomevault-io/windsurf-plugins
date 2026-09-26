---
trigger: always_on
description: You build new components in isolation. Your job ends when the component works. Integration into the website is handled by a separate agent.
---

# Builder Agent Rules — Standalone Components

You build new components in isolation. Your job ends when the component works. Integration into the website is handled by a separate agent.

## Scope of this file

**This file covers STANDALONE components only.** Everything in `components-workspace/` is a standalone — an experimental, creatively-free component that lives alongside ~40 others on the homepage grid. Standalones follow the rules in this file.

If your brief has `design-system: andromeda` (or `meridian`, etc.) — you are NOT working on a standalone. Stop reading this file and go to [`../design-systems/CLAUDE.md`](../design-systems/CLAUDE.md). Design-system work has a completely different ruleset: token-locked, no creative freedom, follow the existing system's patterns exactly.

If your brief has `design-system: standalone` (or omits the field), continue.

**`tier: premium`?** This is a closed-source standalone. Build and preview it here exactly like any other standalone — same rules in this file — but tag every file with the premium marker and store the finished source in the private `aicanvas-premium` repo; it must never be committed to this public repo. Read `_private/premium-workflow.md` (gitignored) for the marker value, the manifest entry, and the `PREMIUM_LOCAL_PATH` preview command.

## Skills library — read before building

### Standalone-specific skills (this folder)

- [`skills/aesthetic-freedom.md`](skills/aesthetic-freedom.md) — your primary creative guide (stub, TODO: flesh out). Permission-giving direction: use any colors/gradients/motion you want; the only hard constraint is container chrome (`bg-sand-100 dark:bg-sand-950`).
- [`skills/promotion-guide.md`](skills/promotion-guide.md) — how to promote a design-system component to a standalone via a re-export wrapper (stub, TODO).
- [`skills/card-stack-pattern.md`](skills/card-stack-pattern.md) — slot-based card stacks: swipe-to-dismiss, fan-out, order cycling, spring constants, dot indicators. Read this for ANY component involving stacked or fanned cards.

### Shared tech skills (root `skills/`)

These live at the project root, one level up from this file:

- `../skills/component-anatomy.md` — required structure, naming conventions, what to avoid
- `../skills/animation-patterns.md` — Framer Motion patterns, spring presets, cleanup patterns
- `../skills/prompts-guide.md` — how to write the 3 platform prompts
- `../skills/tailwind-v4.md` — Tailwind v4 patterns, what is different from v3
- `../skills/typescript-patterns.md` — TypeScript patterns, event types, MotionValue types

### Site design tokens (when you need them)

- `../supervisor/skills/site-design-tokens.md` — the main AI Canvas sand/olive/Manrope system. You rarely need this for a standalone's internals (creative freedom), but the **container chrome** uses sand-100/sand-950 from this file, and if a standalone deliberately echoes the site's look you can reference it here.
- `../supervisor/skills/seo-metadata.md` — SEO rules for writing the `description` field. The description you write in `spec.md` and the registry entry becomes the component's Google meta description. Read this before writing any description.

## Extended skills (`.claude/skills/`) — read for complex visual components

- **`.claude/skills/creative-3d-components/SKILL.md`** — Read this for ANY 3D, particle, canvas, or visually complex component. Contains specific visual recipes (glass, glow, liquid, wireframe), creative direction, color palettes for dark backgrounds, mathematical patterns (noise, parametric curves, springs), and interaction patterns. This is not API docs — it's opinionated creative guidance with code.
- **`.claude/skills/design-motion-principles/SKILL.md`** — Read when tuning animation quality. Perspectives from three motion designers (Emil Kowalski, Jakub Krehel, Jhey Tompkins) with context-aware recommendations.

## Your scope
- Build inside `components-workspace/<component-name>/` only
- Create `index.tsx` (the component) and `prompts.ts` (3 platform prompts: Claude Code, Lovable, V0 — any subset applicable to the component)
- Do NOT touch `app/`, routing, the navbar, or any website logic

## Component requirements

### index.tsx
- `'use client'` at the top (always)
- Export a single **default** function that matches the folder name in PascalCase (e.g. folder `glowing-button` → `export default function GlowingButton()`)
- Add a `// npm install` comment after `'use client'` listing external npm packages (e.g. `// npm install framer-motion @phosphor-icons/react`). Omit for React-only components. This gives users a copy-pasteable install command.
- Root element must fill the viewport standalone: `className="flex min-h-screen w-full items-center justify-center bg-sand-100 dark:bg-sand-950"` — use `min-h-screen` (not `h-full`) so the component works when copy-pasted into any project without a parent height chain

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aicanvas-me/aicanvas](https://github.com/aicanvas-me/aicanvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
