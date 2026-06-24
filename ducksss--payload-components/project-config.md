---
trigger: always_on
description: Read this first. It's the single source of truth for agents working in this repo. `CLAUDE.md` points here. Humans: see also `README.md` and `CONTRIBUTING.md` (this file links, it doesn't repeat setup).
---

# Payload Components — Agent Guide

Read this first. It's the single source of truth for agents working in this repo. `CLAUDE.md` points here. Humans: see also `README.md` and `CONTRIBUTING.md` (this file links, it doesn't repeat setup).

## What this is

Payload Components is an **open-source, community-first** registry + CLI that installs Payload CMS blocks into consumer **Payload v3 + Next.js** projects — _wired, not pasted_. A plain `shadcn add` only copies files; `payload-components add` also does the Payload wiring (registers the block in the collection, maps the renderer, regenerates types + the admin import map) and lands it as a reviewable git diff.

This repository is **two things at once**:

1. The **Fumadocs-powered Next.js site** — the marketing landing, docs, and component catalog (this is what's deployed).
2. The **`payload-components` registry + CLI** — the tooling that distributes components into _other_ people's repos.

It is **not** a Payload CMS runtime app: no admin, no database, no `PAYLOAD_SECRET` for the site.

## Project positioning — community-first

The North Star is **open-source, MIT, community-first**. No pricing, no license keys, no gated tiers, no sales funnel. Success is adoption and contributions, not revenue — the catalog grows from real installs and PRs. Keep all copy, docs, and issue templates aligned to that. If you find commercial / "sellable" framing (pricing hedges, "design partner" or "early access" funnels, paid-tier roadmaps), treat it as stale residue and reframe it community-first.

## The mental model (internalize this one split)

|            | Site runtime                  | Component distribution                              |
| ---------- | ----------------------------- | --------------------------------------------- |
| Lives in   | `src/`, `content/docs/`       | `payload-components/`, `tools/payload-components/`         |
| Runs       | here (Fumadocs / Next.js)     | in the _consumer's_ repo, after install       |
| Payload?   | none (docs site only)         | the installed component code **is** Payload target code |

The Payload block code under `payload-components/source/` is **target code** — shipped into consumer repos, never executed by this site. Don't import it into the site, and don't run the site against a database.

## Repo map

| Path | Purpose |
| --- | --- |
| `src/app/` | Routes: `/` (landing), `/docs/[[...slug]]` (Fumadocs), `/components` (catalog), `/about`, `/api/search`, `llms.txt` · `llms-full.txt` · `llms.mdx` (AI surfaces), `og/` + `opengraph-image` |
| `src/components/site/` | Site UI: `SiteHeader`/`SiteFooter`, `HeroProductFrame` + `HeroInstallReplay` (the install replay), `WiringLedger`, `ComponentSpecimen`/`ComponentCard`/`ComponentGrid`, `Faq`, `CommandCopyButton`, `section.tsx` |
| `src/components/site/sections/` | Landing sections (Hero, StackBand, Tax, Workflow, Wiring, Catalog, Faq, CommunityCta). `src/app/page.tsx` just orchestrates these |
| `src/components/site/demos/` | **Demo twins** — live previews that mirror component source class-for-class (see Core flows) |
| `src/lib/site.ts` | **Single source of truth for all site copy/data** (hero text, FAQ, component entries, landing-section headings, terminal demo lines). Tests import from here |
| `content/docs/` | Fumadocs MDX (index, architecture, installation, registry, contributing, `components/*`); page tree via `meta.json` |
| `payload-components/` | `registry.json` (source shadcn registry), `source/` (component target code), `manifests/*.json` (wiring contract), `schema/`, `support-matrix.json` |
| `tools/payload-components/` + `bin/payload-components.mjs` | The CLI: `add` command, project detection, fragment patching, install state, registry build/check |
| `tests/` | `e2e/` (Playwright) + `int/` (Vitest) — the contract (below) |
| `public/r/` | Generated public registry — **gitignored build output**, never hand-edit |

## Core flows

**`payload-components add <component>` pipeline** (`tools/payload-components/commands/add.ts`) — five idempotent stages; install state is recorded in the consumer's `.payload-components/state.json`:

`registry-build` → `registry-add` → `dependency-install` → `fragment-apply` → `post-install` (`generate:types`, `generate:importmap`)

Fragment patching is **text-anchor based** — it finds anchors like `const blockComponents = {` and `name: 'layout'` in the consumer repo and inserts imports/registrations with dedup checks. Fragile by design for now; keep the anchors and dedup logic intact.

**Two install modes:**

- _payload-components-required_ page blocks (`hero-basic`, `feature-grid-basic`) — need the full wiring above.
- _shadcn-native_ post components (in development) — file-only `shadcn add`, no Payload wiring.

**Registry generation:** `payload-components/registry.json` → `public/r` via `shadcn build` (`pnpm registry:build`); `pnpm test:registry` checks the generated output is reproducible.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ducksss/payload-components](https://github.com/Ducksss/payload-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
