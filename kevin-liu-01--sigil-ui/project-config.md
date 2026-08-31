---
trigger: always_on
description: > Read this before writing any code in a Sigil project.
---

# Sigil UI — Agent Instructions

> Read this before writing any code in a Sigil project.

## The #1 Rule

**Edit the token spec. Not the components.**

Sigil is a token-driven design system. Every visual property — colors, fonts, spacing, radius, shadows, motion, borders — flows from a central token spec through CSS custom properties into components. When you need to change how something looks, you change the tokens. The components update automatically.

```
WRONG — manually editing component styling:
  open Button.tsx → find bg-indigo-600 → change to bg-emerald-600
  open Card.tsx → find rounded-lg → change to rounded-none
  open Input.tsx → find border-gray-300 → change to border-gray-500
  (repeat for every component, miss some, drift, inconsistency)

RIGHT — editing the central token spec:
 change --s-primary → every primary-colored element updates (buttons, links, focus rings, badges, gradients)
 change --s-radius-md → every medium-radius element updates (cards, inputs, dropdowns, tooltips)
 run "sigil preset anvil" → the ENTIRE visual identity changes in one command
```

This is the fundamental insight. Agents that understand this build 10x faster because one token edit replaces dozens of component edits.

## How It Works

```
DESIGN.md (human/agent-editable markdown — all 33 categories, 519 tokens)
 ↓ parseDesignMarkdown() / sigil design compile
SigilTokens (TypeScript object, 519 configurable fields)
 ↓ compileToCss() / compileToTailwind() / compileToW3CJson()
CSS custom properties (--s-primary, --s-radius-md, --s-duration-fast, ...)
Tailwind v4 @theme block
W3C Design Tokens JSON
 ↓ consumed by
350+ Components (read var(--s-*), never hardcode values)
```

Legacy path (still supported): `sigil.tokens.md` → `parseMarkdownTokens()` → 8 core groups.

To change the visual output, edit the top of this chain — not the bottom.

## Quick Reference: What to Edit for Common Tasks

| Task | What to edit | Do NOT edit |
|------|-------------|-------------|
| Change primary color | Token CSS: `--s-primary: oklch(...)` | Component files |
| Change fonts | Token CSS: `--s-font-display: "NewFont", ...` | Component files |
| Change all border radius | Token CSS: `--s-radius-md: 12px` | Component files |
| Change animation speed | Token CSS: `--s-duration-fast: 200ms` | Component files |
| Change card hover effect | Preset: `cards.hover-effect: "glow"` | Card component file |
| Change button press scale | Preset: `buttons.active-scale: "0.95"` | Button component file |
| Change background pattern | Preset: `backgrounds.pattern: "dots"` | Layout files |
| Change hero padding/layout | Token CSS: `--s-hero-padding-y: 120px` | Hero component files |
| Change CTA layout | Token CSS: `--s-cta-layout: "split"` | CTA component files |
| Change footer structure | Token CSS: `--s-footer-columns: 3` | Footer component files |
| Change page density | Preset: `pageRhythm.density: "editorial"` | Section/layout files |
| Complete visual overhaul | `npx @sigil-ui/cli preset <name>` | Any component files |
| Targeted brand update | Edit `sigil.tokens.md` or token CSS | Scattered Tailwind classes |

## Repository Structure

```
packages/
  tokens/           @sigil-ui/tokens     — Source of truth: types, defaults, compiler, markdown parser
  presets/           @sigil-ui/presets    — 46 curated preset bundles (44 named + default + template, 519 tokens each)
  components/        @sigil-ui/components — 350+ token-driven React components (read from tokens, don't hardcode)
  primitives/        @sigil-ui/primitives — 40+ headless behavior primitives (Radix UI + Base UI)
  cli/               @sigil-ui/cli       — CLI: init, add, preset, diff, doctor
  create-sigil-app/                      — npx create-sigil-app bootstrapper

apps/
  web/               Product site + docs + sandbox (Fumadocs at /docs, drag-and-drop component canvas, live preset switching)
  demos/             17 demos: 10 templates + 7 showcase clones (ai-saas, dashboard, portfolio, ecommerce, blog, agency, cli-tool, startup, dev-docs, playground, vercel-clone, linear-clone, vite-clone, viteplus-clone, dedalus-clone, oxide-clone, voidzero-clone)

style/               Design language and engineering philosophy (from Dedalus)
  design.md          Design & animation guide: when to animate, speed, seven rules, enter/exit
  ux-principles.md   Product UX rules: 100ms budget, three-click nav, visual design, interaction
  style.md           Engineering philosophy: code quality, YAGNI, hard limits, React/TS rules

skills/              Agent skills for specific workflows
  sigil-tokens/      Edit/extend tokens
  sigil-preset/      Create/modify presets
  sigil-component/   Create/modify components
  sigil-layout/      Page layout with grid system
  sigil-playbook/    Page composition (10 rules from Reticle design language)
  sigil-migration/   Migrate from shadcn/ui
  sigil-polish/      Interface polish: typography, surfaces, animations, performance (5 files)
  sigil-design/      Generate, parse, and compile DESIGN.md
  sigil-messaging/   Canonical positioning, copy, stats, naming, tone
  sigil-audit/       Browser/static audits (scripts/audit-*.mjs against `next start`)
  sigil-scene/       Scene file authoring convention (*.scene.tsx for animated blocks)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kevin-Liu-01/Sigil-UI](https://github.com/Kevin-Liu-01/Sigil-UI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
