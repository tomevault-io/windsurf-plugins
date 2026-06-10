---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## What this is

A research harness + Next.js viewer for the study **"Who Are You? — Cross-Vendor Identity Confusion in Frontier LLMs."** It asks the same question ("Who are you?") to many vendors' frontier models, in 10 languages, N times each, then uses an extractor model to label which vendor each answer *claims* to be, and aggregates the cross-vendor confusion into a graph + report. All model calls go through **ZenMux's Anthropic Messages endpoint** (`https://zenmux.ai/api/anthropic`) via the `@anthropic-ai/sdk` client.

The relationship graph is **rendered and exported only from the web viewer** (the graph studio at `/research/studio`, via the `/api/export` route) — there is no CLI render step. The pipeline stops at `aggregate.json`; everything visual (graph PNG/SVG, image export) is driven by manual interaction in the browser.

## Commands

```bash
export ZENMUX_API_KEY=sk-...   # required by loadConfig; scripts abort without it

# Data pipeline (deliberately separate so you inspect data before writing the report):
pnpm study:test       # run → extract → aggregate (chained, with completeness gate)
pnpm study:report     # aggregate.json → report.md

# Individual steps:
pnpm study:run        # ask pass only (auto-retry rounds + resume)
pnpm study:extract    # identity-extraction pass only (needs complete records)
pnpm study:aggregate  # join + summarize only (needs complete records)

# Pool several runs gathered in stages into one merged result (no API calls):
pnpm study:mix --runs <stampA,stampB,...>   # or --all to pool every native run
pnpm study:aggregate --run mix-<stamp>      # then aggregate/report the mix as usual

# Web viewer (also where the graph is rendered + exported as PNG/SVG):
pnpm dev              # http://localhost:3000/research  ·  /research/studio  ·  /research/browse
pnpm build && pnpm start
pnpm lint             # eslint (flat config, eslint-config-next)
```

There is **no test runner** — `study:test` is the data pipeline, not a unit-test suite. `pnpm` is the package manager (README uses it throughout). The graph is **not** rendered from the CLI; open the studio and export from there.

### Common script flags
- `--config <path>` — config file (default `config/study.yaml`)
- `--run <stamp|latest>` — resume an existing run directory; `study:run` without it creates a fresh timestamped run, the others default to `latest`
- `study:run` only: `--model-concurrency <n>`, `--batch-size <n>`, `--max-rounds <n>` (default 5)
- `study:extract`/`study:aggregate`: `--force` to bypass the completeness gate; `study:extract --re-extract` to redo all extractions
- `study:mix`: `--runs <stamp,stamp,…>` (comma-separated source stamps) **or** `--all` (every native run, skipping prior `mix-*` dirs). Writes a new `mix-<stamp>/` dir; never resumes/overwrites.

## Use the installed skills — don't hand-roll what a skill owns

This repo vendors a set of agent skills (`.Codex/skills/` → symlinks into `.agents/skills/`, pinned in `skills-lock.json`). They are not optional reading — for the matching task, **invoke the skill first** rather than writing UI/animation/Next.js code from memory. The skill carries the current, version-correct conventions; your training data may be stale.

| When you are about to… | Invoke | Why |
|---|---|---|
| Add / change a **shadcn component** (anything under `src/components/ui/`, or `shadcn add`) | `/shadcn` | This project has `components.json` (style `radix-nova`, base `neutral`, `radix-ui` + `lucide`). The skill knows the registry/MCP, correct `add` flow, and how to compose/debug — never copy-paste a component by hand. |
| **Design or beautify** any page/component (`/research`, `/research/studio`, root `page.tsx`, the OG image) | `/ui-ux-pro-max` | Color systems, font pairing, layout, spacing, interaction states, accessibility for the exact stack (Next.js + Tailwind + shadcn). Use it to *plan* before building and to *review* after. |
| Build a **distinctive new surface** from scratch (landing/hero, a poster, a fresh page) | `/frontend-design` | Production-grade, non-generic visual design — pairs well with `/ui-ux-pro-max`. |
| **Audit accessibility / UX** of UI you just wrote | `/web-design-guidelines` | Checks against the Web Interface Guidelines (a11y, semantics, states). |
| Touch **Next.js conventions** (RSC vs client boundary, `force-dynamic`, metadata, route handlers, `next/image`) | `/next-best-practices` | The viewer is Next.js 16 / React 19; the studio/browse pages lean on RSC + `force-dynamic`. |
| **Optimize React/Next perf** (re-renders, data fetching, bundle, server-serialization like browse's "only selected model") | `/vercel-react-best-practices` | Performance patterns specific to this stack. |
| Build any **animation / motion / video** | `/remotion-best-practices` | Remotion + React motion conventions. (No Remotion in the repo yet — reach for this if you add any.) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZenMux/zenmux-arena](https://github.com/ZenMux/zenmux-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
