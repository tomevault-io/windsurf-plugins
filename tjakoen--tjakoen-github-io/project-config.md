---
trigger: always_on
description: How we build on this stack. The goal is one consistent, reusable, scalable way to do
---


How we build on this stack. The goal is one consistent, reusable, scalable way to do
each thing — so new code (and new sessions) extend the grain instead of fighting it.
When a rule and the surrounding code disagree, the surrounding code wins until this doc
is updated; keep them in sync.

> Companion docs: [`PHILOSOPHY.md`](https://github.com/tjakoen/tjakoen.github.io/blob/main/docs/PHILOSOPHY.md) (the why), [`ARCHITECTURE.md`](ARCHITECTURE.md)
> (the substrate), [`GRAIN.md`](../../grain/docs/GRAIN.md) (the design system + AI layer),
> [`AI-INTERFACE.md`](../../grain/docs/AI-INTERFACE.md) (the contract),
> [`DESIGN-SYSTEM.md`](../../grain/docs/DESIGN-SYSTEM.md) (the visual identity), [`grain/README.md`](https://github.com/tjakoen/grain/blob/main/README.md) (usage).

---

## 1. Layers & boundaries

Four concerns, one direction of dependency (each layer builds only on those below):

```
batch/   the no-build hypermedia substrate (render, http, assets, catalog, platform)
   └─ grain/   the design system + optional AI-interaction layer (default theme lives here)
        ├─ mill/       the Markdown→GRAIN CMS (a reusable layer above grain; built)
        └─ tjakoen.github.io/  THE app + composition root — a custom BATCH+GRAIN site that uses
                               MILL for content; holds domain components, routes, pages, server.ts
   (project/   the AI-assistant product — PAUSED since 2026-07-05, a docs-only archive)
```

**Hard rules (enforced by review; verified in the audit):**
- `batch/` imports **nothing** from `grain/` (or the app). It's the substrate; it must extract cleanly.
- `grain/` imports **nothing** from `batch/`. It depends only on the **`OpChannel` port**
  (`grain/ai/contract.ts`) — never a concrete substrate. It ships its own default theme.
- `tjakoen.github.io/` wires the graph. Cross-layer dependencies are declared as **constructor/factory
  params**, and the **only place the layers meet is `tjakoen.github.io/server.ts`** (the composition root).
- New design-system work goes **in `grain/` by default** (it's reusable). Only obviously
  app-specific things (a one-off page layout, a domain component like `task-card`) live in the app
  (`tjakoen.github.io/`). Test: *"would another product on GRAIN want this?"* → yes = grain, no = the app.

A consuming product **re-skins by overriding token slots** in its own sheet linked after
GRAIN's three (`variables.css` → `global.css` → `grain.css`) — never by editing components.

**What "no-build / native-first" governs (and what it doesn't).** The constraint is about the
*product's runtime*, not the dev toolbox. It means exactly two things: (1) **no build step** — Bun
runs the TypeScript directly, no bundler/transpiler between source and server; (2) **native-first**
— the product ships (near-)zero framework JS to the browser (the `bun run audit` numbers are the
proof). It does **not** mean "zero dependencies." Two things are always fair game and are **not**
violations: **platform builtins** (`fs`, `path`, `node:fs/promises` — provided by Bun; batch reads
files with them throughout) and **devDependencies** used by tooling that never ships to the client
(`@playwright/test` drives the e2e tests, `bun run shots`, and `bun run audit` — it measures the
product from the outside, it isn't part of it). The bar to defend is the `dependencies` block in
`package.json`: keep third-party *runtime* deps at zero (today only `bun` itself). A dev tool
importing playwright, or the substrate importing `fs`, is the stack working as intended.

**Native-first is a *positive* rule, not just an absence of framework JS.** It means: **prefer the
platform's own primitive over reimplementing it.** A `<dialog>` over a JS modal; `<details>` over a
JS accordion; the **View Transitions API** over a JS page-animation lib; **plain `<a>` + CSS** over a
JS tab/router; **native constraint validation** over JS form validators; the **Popover API** and
**CSS anchor positioning** over a floating-UI library; **`:has()`** / **`:focus-within`** /
**`color-mix()`** / **`@starting-style`** / **container queries** / scroll-driven animations over
style-computing JS. The test when reaching for JS: *does a browser primitive already do this?* If
yes, use it; only write JS when none does (in this stack the sole such case is the `/intent`
dispatcher). GRAIN's running inventory of which primitives are in use is in
[`grain/docs/GRAIN.md`](../../grain/docs/GRAIN.md) ("What GRAIN gives you"); page transitions are
worked in [ARCHITECTURE §11.3](ARCHITECTURE.md).

---

## 2. TypeScript

- **Factories, not classes**, for wiring: `createX(deps)` / `makeX(opts)` returning a small
  object of closures (e.g. `createInteractionLayer`, `makeStubReasoner`, `createStream`).
  Classes are reserved for **port implementations** (`InMemoryItemRepository implements
  ItemRepository`), domain **error types** (`HttpError`), and plain **service aggregators**.
- **Depend on interfaces, not implementations.** Every cross-layer seam is a named `interface`
  (`OpChannel`, `Reasoner`, `ReasonTools`, `ItemRepository`, `Runtime`, `Stream`). Inject the
  concrete thing at the composition root.
- **Erasable TypeScript only** (`erasableSyntaxOnly` + `verbatimModuleSyntax`): no `enum`, no

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tjakoen/tjakoen.github.io](https://github.com/tjakoen/tjakoen.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
