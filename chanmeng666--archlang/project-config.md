---
trigger: always_on
description: Guidance for Claude Code (and any AI agent) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (and any AI agent) working in this repository.

The **canonical, always-current** project status, architecture, commands, and conventions live in
**[AGENTS.md](AGENTS.md)** — read it first. It is imported below so it loads with this file; for the
exact shipped state and versions, defer to AGENTS.md → "Project status" and `CHANGELOG.md` rather
than memory.

@AGENTS.md

## Orientation (the rest is in AGENTS.md)

- **What this is:** ArchLang — a small declarative language that compiles `.arch` floor-plan source
  to professional **SVG** (also DXF/PDF/PNG). Pure TypeScript, **zero runtime dependencies**,
  isomorphic (runs in Node and the browser). A published, deployed monorepo, not a WIP.
- **Build & run:** `npm run build` · `npm test` (vitest) ·
  `npm run cli -- compile examples/studio.arch -o out.svg`. A single root `npm install` bootstraps
  every workspace.
- **Brand:** the logo is an "A" drawn as an A-frame house floor plan.
  `brand/archlang-logo-master.svg` is the byte-sacred source — every variant is a **fill-swap only**
  (never re-trace/simplify/re-fit path data, no small-size tier). The two public sites run the shared
  **"The Compile Boundary"** design system — a cool source surface and a warm sheet surface split by a
  compile seam, **both LIGHT: there is no dark mode and no dark surface on either site**. Its token
  block is **duplicated byte-identically** in `docs-site/.vitepress/theme/style.css` and
  `playground/src/styles/tokens.css` (change one, change the other). See
  [ADR 0014](docs/adr/0014-one-light-world.md) — which supersedes
  [ADR 0010](docs/adr/0010-compile-boundary-design-system.md) §1/§2/§6/§7, so read 0010's carbon/mylar
  prose as history — and `brand/README.md` first.

## Non-negotiable invariants (break these and CI fails)

- **`compile()` is pure, synchronous, deterministic.** No I/O, no `Date.now()`, no `Math.random()`
  in `src/` core; output is byte-for-byte stable and snapshot/golden-tested. Node APIs and real time
  are allowed **only** in `src/cli.ts` + `src/cli/`; everything else gets its environment via the `World` seam.
  Route number formatting through `fmt()` so floats don't drift. The parse-stage memo's `PlanNode`
  is shared and must never be mutated downstream — clone before mutating (a `repair()` in-place edit
  made output history-dependent; fixed in `51a47ee`).
- **Don't hand-edit generated files.** `dist/`, `editors/*.tmLanguage.json`,
  `playground/src/arch-language.js`, `docs-site/.vitepress/theme/arch-highlight.js`,
  `docs/error-codes.md`, `docs/cli-reference.md`, `spec.llm.md`,
  `llms-full.txt`, `schemas/plan.schema.json`, `schemas/intent.schema.json`,
  `grammars/archlang.gbnf`, and the twenty committed `examples/*.svg` the README embeds (the
  `README_SVGS` list in `scripts/gen-example-svgs.ts`) are generated — edit the source
  (`src/grammar/tokens.ts`, `src/error-catalog.ts`, `src/manifest.ts`, `PLAN_JSON_SCHEMA`,
  `INTENT_JSON_SCHEMA`, `examples/`, `SKILL.md`) and run the matching `npm run gen:grammars` /
  `gen:errors` / `gen:cli` / `gen:spec` / `gen:llms` / `gen:plan-schema` / `gen:intent-schema` /
  `gen:gbnf` / `gen:example-svgs`. CI fails on drift. The SVGs joined this list late and are the
  clearest case for it: hand-committed and never re-rendered, three of them showed the README a
  building compiled before four separate rendering fixes, and nothing was watching.
- **A generator's TEMPLATE can go stale even when `check:drift` is green.** The gate compares
  generator *output* to the committed file — it proves reproducibility, not correctness. A generator
  that hardcodes a language fact reproduces the same *wrong* text forever: `gen-llm-spec.ts` shipped a
  v1.12 CLI + no `strip` for three releases, and `gen-grammars.ts` hardcoded a number regex without the
  unit suffixes. **Derive from the source of truth (`KEYWORDS`/`RULES`/`buildManifest()`), never
  retype it**, and give each generator a guard that fails when a source-of-truth entry has no
  rendering (as `gen-llm-spec.ts` now does for every `KEYWORDS.control` entry, not just `element`).
- **A derived POSITION comes from the shape, never from its bounding box or centroid.** Six silent
  bugs shipped this way and were fixed in v1.25.0 — a label drawn off its own floor, a walk reported
  at half its true length, witness lines hanging metres off a sloped facade, a door swung into a wall
  its room does not touch, a fixture backed onto a wall outside its room, and every courtyard-wall
  window facing backwards. **`arch lint` reported none of them.** The grep that finds the next one is
  `room.size`/`r.size.w` with no nearby `r.poly` branch. Fix locally and in closed form — probe one
  wall thickness off each face and ask which side has floor — and never reach for the wall boolean
  union to answer a `describe()` question. Inventory: `docs/research/2026-08-06-competitor-borrowing-roadmap.md` §9.1.
- **Every new language form ships with a byte-identity law, pinned by test:** a plan that does not
  use it renders, describes and lints exactly as before. `site`, the door kinds, `zone`, `paper`,
  `polygon`, `arc`, `roof` and `void` all have one. Prove it with a SHA-256 sweep over the shipped

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChanMeng666/archlang](https://github.com/ChanMeng666/archlang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
