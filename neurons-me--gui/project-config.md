---
trigger: always_on
description: Agent context for the monorepo as a whole lives in `all.this/AGENTS.md` — read
---

# Agent context — this.gui

Agent context for the monorepo as a whole lives in `all.this/AGENTS.md` — read
that one first for the mesh/NRP argument and cross-package state. This file is
the GUI-specific counterpart: what this package *is*, how it's put together
internally, and the landmines specific to it. `CLAUDE.md` (repo root) has the
authoritative build/dev commands — this file does not repeat them.

If you only have this repo checked out standalone (not nested inside
`all.this`), find the parent at the `all.this` repo under the `neurons-me`
GitHub org.

## What this.gui is

`this.gui` ("Generative User Interfaces") is a React component library with
**two authoring paths that share one component surface**:

1. **React-first (JSX)** — import components and compose them directly, like
   any component library. This is the path most app code should use.
2. **Declarative (spec tree)** — build a `GuiSpecNode` (`{ type, props,
   children, provenance }`, plain data, not a React element) and hand it to
   `renderNode()` / `mount()` / `SpecBoundary`. This is the path for
   LLM-generated UI, data-driven admin panels, and anything that needs the
   Semantic Inspector to see every node. `type` can be a string (resolved
   against a registry) or a real component reference.

Both paths render the *same* underlying components — a spec node with
`type: 'Button'` and a `<Button>` JSX element end up rendering the same
`Button.tsx`. What differs is who builds the tree and whether nodes get
registered for inspection.

Composition contract (see `README.md` for the full walkthrough):

```
Theme → Layout → view
```

`Theme` is the visual environment (tokens, mode). `Layout` is the chrome —
`TopBar` / `LeftBar` / `RightBar` / `Footer`. Your view fills the content
area. Components are organized **atoms → molecules → compounds**: primitives
compose into patterns, patterns compose into features.

## Package surface (subpath exports)

Each subpath is a separate Vite lib entry (see `vite.config.js`, `entry: {...}`)
with its own ES + CJS output — importing one does not pull in the others.

| Entry | Source | What it gives you |
|---|---|---|
| `this.gui` | `index.ts` | `Theme`, `Layout`, `ThemeLauncher`, and most top-level components (explicit exports only — see rule below) |
| `this.gui/atoms` | `src/gui/Atoms/atoms.ts` | Primitives — `Box`, `Button`, `Typography`, `Icon`, … |
| `this.gui/molecules` | `src/gui/Molecules/molecules.ts` | Compositions — `Hero`, `Stack`, `Page`, … |
| `this.gui/compounds` (alias `/components`) | `src/gui/Compounds/compounds.ts` | Feature panels built from molecules |
| `this.gui/react` | `src/react-entry.ts` | `.me` bridge — `MeRuntimeProvider`, `useMeValue`, `useMeAction` |
| `this.gui/runtime` | `src/runtime-entry.ts` | `mount`, `mountApp`, `declareApp`, `createMeRuntime`, `writeMeValue`/`readMeValue`, `renderNode` |
| `this.gui/devtools` | `src/devtools-entry.ts` | Semantic Inspector / admin view, opt-in only |
| `this.gui/cleaker` | `src/cleaker-entry.ts` | Framework-free X-Me-Proof signing (`signedRequest`, `deriveCleakerNode`, `fetchGatewayHostname`) |
| `this.gui/legacy` | `src/legacy/index.tsx` | UMD/global compatibility facade |
| `this.gui/style.css` | `dist/styles.css` | Compiled stylesheet |

**Root barrel rule** (`index.ts`, stated at the top of the file): no
`export *`. Every root export is explicit, and pulled from a concrete module
path (`@/gui/Atoms/Button/Button`), never re-exported through a barrel. This
keeps the root entry tree-shakeable and its public surface intentional. When
adding a new root export, follow the existing pattern — explicit named
export, concrete import path, not a wildcard.

## Two build outputs, one source

This package ships **both** an ESM/CJS npm package (multi-entry, tree-shaken,
for bundler consumers) **and** a single-file UMD/IIFE bundle (for `<script>`
tag consumption with global `window.React`/`window.GUI`, used by
`netget`'s static `namespace-surface` HTML shell and by `monad`'s HTML
bootstrap). Same `index.ts` root, two build passes (`UMD_TARGET=core` vs the
multi-entry pass) — see `vite.config.js`. React/ReactDOM stay external peer
deps in both modes; the UMD build additionally bundles router + JSX runtime
so it works standalone without extra global shims.

**This dual consumption is a real constraint on how you write root-barrel
code**, not just a build detail — see the react-reconciler landmine below.

## Declarative runtime — the pieces, bottom to top

- **`GuiSpecNode`** (`src/types/gui.types.ts`) — the data shape:
  `{ type, props?, children?, parts?, provenance? }`. `provenance` carries
  `semanticPath` / `explainPath` / `source` / `note` — what the Semantic
  Inspector's "Explain" panel shows for a node. A node with no `provenance`
  shows as "Not declared", which is fine for structural chrome but should be
  filled in for anything that reads/writes `.me`.
- **`renderNode()`** (`src/runtime/renderer.ts`) — the low-level spec → React
  element function. Resolves `type` (string via registry lookup, or a direct
  component/element/Fragment-like symbol), resolves `props` through a
  `RuntimeAdapter` (below), and — if `onNodeResolved` is passed — schedules a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neurons-me/GUI](https://github.com/neurons-me/GUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
