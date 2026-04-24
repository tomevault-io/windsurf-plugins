---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Planet Command Design** is a game design tools & calculators site for the "Planet Command" universe. It provides interactive tools for spacecraft engineering: struct memory sizing, power generation design, production chain logistics, and physics calculations (projectiles, lasers, particle beams).

## Build & Dev Commands

```bash
npm run dev          # Runs Vite dev server + Express backend concurrently
npm run dev:client   # Vite dev server only
npm run dev:server   # Express backend only (tsx watch)
npm run build        # svelte-check && vite build
npm run preview      # Preview production build
```

The dev server proxies `/api` requests to `http://localhost:3001` (Express backend).

There is no test runner or linter configured. `npm run build` runs `svelte-check` for type-checking before the Vite build.

## Directory Structure

```
├── .github/workflows/deploy.yml   # CI/CD: auto-deploy on push to main
├── server/                         # Express backend (port 3001)
│   ├── index.ts                    # Entry point, CORS, JSON middleware
│   ├── routes/save.ts              # POST /api/save — JSON file persistence
│   ├── tsconfig.json               # Backend TS config (ES2022, Node)
│   └── data/                       # Runtime JSON storage (gitignored)
├── src/
│   ├── main.ts                     # Svelte app mount point
│   ├── App.svelte                  # Root layout, tab bar, hash routing
│   ├── types.ts                    # ToolDef interface
│   ├── styles/global.css           # Dark theme CSS custom properties
│   ├── components/
│   │   └── SliderInput.svelte      # Reusable slider+number input (linear/log)
│   └── tools/
│       ├── struct-sizer/           # C++ struct memory calculator
│       ├── power-gen/              # Power generation & thruster design
│       ├── production-chain/       # Spacecraft construction logistics
│       └── calculations/           # Physics calculators (projectile/laser/particle beam)
├── index.html                      # HTML entry point
├── package.json                    # ES module, scripts, deps
├── tsconfig.json                   # Frontend TS config (ES2020, DOM)
├── vite.config.ts                  # Vite + Svelte plugin, /api proxy
└── svelte.config.js                # Svelte preprocessor config
```

## Architecture

**Svelte 5 + Vite + TypeScript** game design tools site with tab-based navigation. ES modules throughout (`"type": "module"` in package.json).

### Frontend

- **Hash routing** — `App.svelte` uses `window.location.hash` (#tool-id) for tab navigation, no router library. Components mount/unmount via `{#key activeToolId}`.
- **Tool registration** — Each tool is a Svelte component in `src/tools/<name>/`. Register by importing the component and adding `{ id, label, component }` to the `tools` array in `App.svelte`. The `ToolDef` interface is in `src/types.ts`.
- **Svelte 5 runes** — Use `$state()`, `$derived()`, `$derived.by()`, `$effect()`, `$props()`, `$bindable()`, `bind:value`, `onclick={handler}`, and direct component references (not `svelte:component`).
- **Styling** — Dark theme via CSS custom properties in `src/styles/global.css`. Fonts: "DM Sans" (body), "JetBrains Mono" (headers/code). No CSS framework. Scoped `<style>` blocks in each component.
- **Charts** — Chart.js v4 with canvas refs (`bind:this`), plus `chartjs-plugin-annotation` for threshold/reference lines. Colors are hardcoded hex constants (CSS vars don't work in canvas). Charts reuse instances via `.update('none')` instead of destroy/recreate for performance. Raw canvas 2D API used for heatmaps (e.g., range-vs-power in laser calculator, emittance feasibility, particle beam heatmaps).
- **Viz panels** — Each chart/visualization is wrapped in a `<details class="viz-toggle">` for collapsible show/hide, with HTML5 drag-and-drop reordering via CSS `order` property (keeps canvas bindings intact since DOM nodes don't move).
- **Reusable components** — Shared UI components live in `src/components/`. Use `SliderInput` for any slider+number input combo (supports linear and `log` mode for large ranges, with optional `inputMax` override).

### Adding a New Tool

1. Create `src/tools/<tool-id>/ToolName.svelte`
2. Optionally add a `data/` subdirectory for JSON reference data
3. Import the component in `App.svelte`
4. Add `{ id: '<tool-id>', label: '<tab_label>', component: ToolName }` to the `tools` array
5. The tool is now routable at `#<tool-id>`

### Current Tools

- **struct-sizer** (`src/tools/struct-sizer/`) — C++ struct memory/padding estimator. Supports 51 types (primitives, STL containers, pointers). No external data files.
- **power-gen** (`src/tools/power-gen/`) — Power generator & thruster design. Sub-tabs for Generators (4 charts) and Thrusters (placeholder). Reference data in `data/*.json` (10 files: generator-comparison, fuel-energy-density, radiator-cooling, radiator-materials, carrier-power-bounds, nuclear-conversion-chain, water-use-nuclear, direct-conversion-methods, fusion-size-factors, fusion-tech-improvements).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bshafer93) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
