---
trigger: always_on
description: Onboarding + operating rules for any AI (or human) joining this repo. Read this first,
---

# CLAUDE.md — start here

Onboarding + operating rules for any AI (or human) joining this repo. Read this first,
then the docs it points to. Keep it accurate: if you change how the project works, update
this file too.

## What this is

A **no-build, server-rendered hypermedia** stack and the things built on it. One direction of
dependency — each layer builds only on the layers below it:

```
batch/   BATCH — the substrate (Bun · Addressable · TypeScript · CSS · htmx); no build step
  └─ grain/   GRAIN — an AI-interaction design system + its default theme (the look) + the catalog
       ├─ MILL/               the markdown CMS (LIVE — renders /notes + the layer docs; its OWN reusable project)
       ├─ proof/              PROOF — the AI plan board, a mountable layer (plans-as-markdown → kanban)
       ├─ pantry/             PANTRY — the installable dev-docs + AI cockpit app (`bunx pantry`)
       ├─ tjakoen.github.io/  THE personal-site app + the composition root
       └─ project/            the product — a personal AI assistant; **PAUSED**, docs-only archive
```

The **composition root folded into `tjakoen.github.io/` (2026-07-05)**: the portfolio is now THE app —
it wires batch + grain + mill and runs the site; the hero desk is the reference surface where you
watch the AI act. The portfolio *uses* MILL for its markdown content; MILL does not build it.
Dependency purity: `grain` imports nothing from `batch` except the `OpChannel` port; **MILL depends
on both, never the reverse** — a new layer above both, not an extension of either.

The defining idea: a UI where **every surface is addressable and operable by both a human
and an AI through one shared vocabulary**, with the AI's presence shown as a visible signal
(*grain = AI*). A human click and an AI decision become the **same `Intent`**, flow through
**one door** (`POST /intent` → `grain/ai/interaction-layer.ts`), and return as **`RenderOp`s**
pushed over SSE. No privileged AI→DOM back channel.

Fuller detail on each layer, and the paused-project history, is in [`DOCS.md`](DOCS.md) — the
full doc map.

## Start here (reading order)

1. **[PHILOSOPHY.md](https://github.com/tjakoen/tjakoen.github.io/blob/main/docs/PHILOSOPHY.md)** — the *why* (the beliefs the whole stack serves). **Read first.**
2. **[CONVENTIONS](https://tjakoen.github.io/batch/docs/conventions)** — the build standard (layering, components, tokens,
   the action vocabulary, the 3-tier testing bar, the extraction plan). **The rulebook.**
3. **[ARCHITECTURE](https://tjakoen.github.io/batch/docs/architecture)** — the substrate's reasoning (single source of truth).
4. **[GRAIN](https://tjakoen.github.io/grain/docs/grain)** + **[AI-INTERFACE](https://tjakoen.github.io/grain/docs/ai-interface)** — the
   design system and the AI contract (surfaces, ops, manifest, the "AI acts" protocol).
5. **[DESIGN-SYSTEM](https://tjakoen.github.io/grain/docs/design-system)** — the visual identity / grade-as-signal.
6. **[`proof/PLAN.md`](https://github.com/tjakoen/grain/blob/main/packages/proof/PLAN.md)** + **[`pantry/PLAN.md`](https://github.com/tjakoen/pantry/blob/main/PLAN.md)** — the plan board and the cockpit app.

The SSOT for what's operable is **`grain/ai/contract.ts`** (`SurfaceKind`, `ActionName`,
`ACTIONS`, `RenderOp`). The composition root — the only place the layers meet — is
**`tjakoen.github.io/server.ts`**. The reference surface is the **hero desk** (the home route `/`),
where a human and the AI drive the same door.

**Working mainly in one layer?** Each layer's repo carries its **own `CLAUDE.md`** —
[`batch/CLAUDE.md`](https://github.com/tjakoen/batch/blob/main/CLAUDE.md) and [`grain/CLAUDE.md`](https://github.com/tjakoen/grain/blob/main/CLAUDE.md) — with that layer's
non-negotiables and the hard-won *"don't repeat these"* lessons. Read the layer's file alongside this
one.

## This repo is the stack's control plane — run it

`bread` is a **map, not a monorepo**, but it is also the one place that operates the *whole stack*
at once. There is no app code to run here; the commands below drive PANTRY against this umbrella host
(its `plans/`, its docs, its layer pins). The per-layer `bun run dev/test/shots/audit` commands live
in each **layer's own repo**, not here.

```bash
bun run cockpit    # bunx pantry serve — the whole-stack cockpit: plans board, decision inbox, docs
bun run check      # bunx pantry check — doc-drift lint (dead references); CI-able, exits nonzero
bun run doctor     # bunx pantry doctor — kit compliance + staleness + layer-pin drift (the omnibus)
bun run deps       # bunx pantry deps — are the @tjakoen/* pins current with the layer sources on disk?
bun run plans:check # bunx proof check — validate the umbrella plan board
bun run deps:refresh # re-pin every layer to its latest (the fix when `deps` reports drift)
```

`deps` / `doctor`'s pin check reads the **sibling layer checkouts** (`../batch`, `../grain/packages/*`):
a pin behind its source is a chore that's **due** (surfaced), never a broken build. Run `deps:refresh`
to clear it. This is the umbrella's unique job — no single layer repo can answer "is the stack pinned
to what I actually have?"

## Non-negotiables (see CONVENTIONS for the full rules)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tjakoen/bread](https://github.com/tjakoen/bread) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
