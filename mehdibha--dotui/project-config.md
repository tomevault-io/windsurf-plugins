---
trigger: always_on
description: This is the dotUI repository — a design-system builder. Users compose a complete design system at dotui.org/create — colors, typography, icons, density, radius, per-component styles — preview every change live on real components, and export it as code they own: into their codebase via the shadcn CLI, or straight into v0 — with Bolt, Lovable, Figma and more planned. It's built on React Aria Components with Tailwind CSS v4 and tailwind-variants, themed by an OKLCH color engine, and distributed as 
---

# dotUI

This is the dotUI repository — a design-system builder. Users compose a complete design system at dotui.org/create — colors, typography, icons, density, radius, per-component styles — preview every change live on real components, and export it as code they own: into their codebase via the shadcn CLI, or straight into v0 — with Bolt, Lovable, Figma and more planned. It's built on React Aria Components with Tailwind CSS v4 and tailwind-variants, themed by an OKLCH color engine, and distributed as a shadcn-compatible registry served from a TanStack Start app.

## Product direction

This is the goal, not the current state — check the code before assuming an axis exists.

The north star: the builder should be flexible enough to recreate almost any design system. If a user can't reproduce the look of a Material-, Geist-, or Linear-style system, an axis is missing. Coverage comes from a complete set of well-chosen axes, not infinite options: **every visual decision is a user-configurable axis of the builder**, never a hardcoded choice. Axes include (not exhaustive):

- Color system: simple or advanced, selectable generation algorithm, semantic tokens, optionally context-aware tokens.
- Typography, icon library, density, radius, interactive/disabled cursors.
- Grouped tweaks — e.g. translucent menus/popovers as a single switch.
- Per-component styles: named variants curated per component — the 20% of styles that cover 80% of design systems — plus hover effect, radius…
- For consistency, related components form synced groups: Button and ToggleButton share the same styles and must stay in sync.

A second customization layer beyond visuals: `codeOptions` — the style of the exported code itself. Separator comments or not, arrow functions vs function declarations, tailwind-variants styles as commented arrays vs one line per slot/variant, etc. The exported design system should read like the user's codebase, not ours.

Beyond that, export keeps widening: CLI + v0 today; Bolt, Lovable, Figma, Claude design, etc. planned.

What this means when writing code today:

- The test for hardcoded values: would two design systems disagree on it? The design system's look (color, radius, typography, shadows, density-affected spacing) goes through tokens/variants — `bg-primary`, not `bg-[#635bff]`. Component mechanics (internal layout, hairlines, hit areas) stay plain values — don't tokenize them. Look with no covering axis? Flag the missing axis; don't invent a token.
- A style change to one component in a synced group is a change to the whole group (Button ⇄ ToggleButton) — land them together.
- New axes and styles must be switchable at runtime (CSS variables, variant props, data attributes), never decided at build time — the builder previews live.
- Registry items import only from `@/registry/*`, relative paths, and published packages — plain React files, shadcn-schema compatible. www-side imports (router, fumadocs, `@/components`) must never leak in.
- Author registry source in one canonical style (current files are the reference) — `codeOptions` will be mechanical publisher transforms over it, and inconsistent source breaks the transforms.

## Current state of the project

An honest snapshot (July 2026) to calibrate against. Task-level work is tracked in GitHub issues; this is the standing picture.

- **Solid:** the docs (one last revision pass pending) and the components — they look and behave well individually. Cross-component composition is less proven: treat "these compose cleanly" as a claim to verify, not an assumption.
- **Known-weak:** the components page (several demos misbehave); the presets page (very slow); the presets themselves on both landing and presets page — due to be replaced by fewer, high-fidelity presets of popular systems (Linear, Vercel, …); chart colors (need a full rewrite); site performance overall (never audited — a docs site must feel instant).
- **Queued for rewrite — don't deepen:** the color system and the entire /create builder experience get complete rewrites — the goal is really good foundations, not incremental fixes. The color rewrite starts with a playground to verify generated colors visually (current output is average); the /create rewrite starts with an experience spec — what should building a design system feel like — before any code. The publisher is also queued (see below), and the registry should get a small refactor and clean up. In these areas, verify behavior by reading the code and keep changes shallow.
- **AI slop:** much of the codebase was written by AI agents and hasn't all been walked through. When working in an area, leave it simpler than you found it — cut needless comments, indirection, and dead options rather than matching the existing style.

## Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mehdibha/dotUI](https://github.com/mehdibha/dotUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
