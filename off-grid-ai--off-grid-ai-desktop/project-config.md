---
trigger: always_on
description: This is **Off Grid AI Desktop** — an Electron (macOS) desktop app. The product name is always **"Off Grid AI Desktop"** (never "Off Grid Desktop", "My Memories", etc.) — in window titles, OAuth client names, about screens, everywhere.
---

# Off Grid AI Desktop — agent guide

This is **Off Grid AI Desktop** — an Electron (macOS) desktop app. The product name is always **"Off Grid AI Desktop"** (never "Off Grid Desktop", "My Memories", etc.) — in window titles, OAuth client names, about screens, everywhere.

## Design — DESKTOP-FIRST, Off Grid brand

Full design doc: **`docs/DESIGN.md`**. The essentials, which OVERRIDE any mobile-first or monochrome assumptions:

- **Desktop-first.** Wide canvas: multi-column layouts, dense lists/tables, side panels, detail screens, hover affordances. Never design mobile-first or for narrow viewports. (The mobile app is a separate product with its own guide.)
- **Typeface: Menlo** (monospace) everywhere — terminal/brutalist.
- **Accent: emerald** — `#34D399` (dark) / `#059669` (light). THE accent for primary actions, active states, links, success. (Tailwind `green-500/400` is an acceptable stand-in but prefer the exact tokens.)
- **Base:** black / `#0A0A0A` + white; neutral grays for surfaces/borders/text tiers. Flat, sharp, dense.
- Tokens: `@offgrid/design`. Brand canon: `mobile/docs/design/DESIGN_PHILOSOPHY_SYSTEM.md` (brand only — desktop *layout* follows `docs/DESIGN.md`, desktop-first).
- Real brand logos (Simple Icons), no decorative tiles behind them; no gradients; no emojis in the UI.

### Use the screen real estate — desktop density rules

The window is WIDE. A list of cards/rows stretched edge-to-edge in a single column (one item per 1900px line, the action button marooned on the far right) wastes the canvas and reads worse, not better. Lay out for the space you have. These are hard rules, learned the hard way on the Models screen:

- **Multi-column responsive grids for collections.** Any list of comparable items (models, connectors, entities, meetings) is a grid that fills the width: `grid-cols-2 lg:grid-cols-3 2xl:grid-cols-4`, not one full-width row each. A card's controls stay next to its content, never flung across empty space.
- **Tight, consistent spacing on a 4/8/12px scale.** Dense data UIs use narrow gutters (8-12px) and small padding, NOT the 16-24px editorial spacing. Body text ~12-14px, compact line-height. Flat and sharp, per the brand.
- **Group, then separate.** Reduce gaps *within* a group (rows in a section) but keep clear separation *between* functional groups (filters vs data, "On this device" vs "Available"). Section headers over a wall of identical rows.
- **Progressive disclosure.** Secondary info and rarely-used controls go behind a detail panel / "…" / hover affordance — don't lay everything flat. Master list stays scannable; depth lives in the side panel or slide-over.
- **Sticky context.** Fix headers, tabs, filter bars, and column labels while the body scrolls, so context never scrolls away.
- **Finesse the interactions.** Every click gets a small micro-interaction — `transition-all duration-150`, `active:scale-95` on buttons, slide+fade (not abrupt mount) for panels/slide-overs. State changes animate; nothing pops in or out hard.
- **Offer density where it matters**, but the default IS dense — this is a terminal/brutalist desktop app, not a spacious mobile-first card feed.

Best-practice references: [UXPin grid systems](https://www.uxpin.com/studio/blog/ui-grids-how-to-guide/), [Pencil & Paper enterprise data tables](https://www.pencilandpaper.io/articles/ux-pattern-analysis-enterprise-data-tables), [Designing for data density](https://paulwallas.medium.com/designing-for-data-density-what-most-ui-tutorials-wont-teach-you-091b3e9b51f4), [Andrew Coyle on large data tables](https://coyleandrew.medium.com/ui-considerations-for-designing-large-data-tables-aa6c1ea93797).

## What this app is

A private, **local-first** layer that **sees** (screen capture → OCR → entities), **remembers** (observations/entities/memory), helps you **reflect** (mind-share / day), and **acts** (MCP connectors + approval-gated actions). Everything is processed on-device by a bundled local LLM (llama.cpp + gemma); nothing routes through a server we own.

Roadmap: **`ROADMAP_DESKTOP.md`** (this repo) and `../shared/ROADMAP.md`.

## Stack

Electron 39 + React 19 + Tailwind v4 + electron-vite; better-sqlite3; bundled `llama-server` (port 8439), `whisper.cpp`, `ffmpeg`, `sharp`. Local LLM is a reasoning model — pass `chat_template_kwargs:{enable_thinking:false}` + grammar-constrained `response_format` for structured output. userData dir is `~/Library/Application Support/Off Grid AI Desktop`.

## Bundled chat engine (`llama-server`) — built in CI, gated, verified the hard way

The chat engine is compiled from llama.cpp **in CI** (`scripts/build-llama.sh`, run by `release.yml` before signing), NOT shipped from the committed/LFS binary. Three engine failures already shipped a broken app to real users — each has a gate now; do not weaken them:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [off-grid-ai/off-grid-ai-desktop](https://github.com/off-grid-ai/off-grid-ai-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
