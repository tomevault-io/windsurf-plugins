---
trigger: always_on
description: Nota is a document language: `@`-syntax markup (after Pollen/Scribble) whose Rust *reader*
---

# Nota — orientation for Claude

Nota is a document language: `@`-syntax markup (after Pollen/Scribble) whose Rust *reader*
compiles `.nota` files **directly to Solid JSX** — a document is a Solid component. Grouping
(paragraphs/lists/sections) happens at runtime in core's `<Reforest>` pass over the rendered
tree; cross-references resolve through the **unified anchor/ref registry** (one doc-state store;
headings/labels/figures/notes/cites are anchors and uses are refs). Facts have opaque, stable
locations assigned in registration order; their snapshot order is separate from identity. SSG
runs a **two-pass render** so forward references converge in the static bytes; interactive pages
hydrate through ordinary Solid hydration. The pre-Solid
architecture (hyperscript `h`/`decode`, islands/replay, the react packages) is deleted;
`design/decode.md` is its archived spec.

## Read first
Most tasks need only this file + the package map. For architecture/emit work:
- **`design/solid.md`** — the current architecture spec (reforest tiers, doc-state, two-pass
  drivers, hydration, the Astro integration).
- **`design/references.md`** — the unified anchor/ref model (`&id` across kinds, note defs
  via `@Note[id]`, resolution-error policy).
- **`design/notation.md`** — surface syntax; its banner scopes the archived emit forms.
- Reader internals live with the code: **`oxc/NOTA_READER.md`**.

## Repo layout
- **`oxc/`** — fork of oxc (branch `nota`) hosting the reader, a **git submodule**. Commit inside
  `oxc/`, then bump the pointer in the main repo. Architecture: `oxc/NOTA_READER.md`.
- **`packages/*`** — the `@nota-lang/*` TypeScript packages, a **Depot** + pnpm workspace:
  - **core** — the Solid runtime: `<Reforest>` (SSR-chunk/DOM tree reconstruction + grouping),
    the doc-state store (opaque fact locations + an ordered snapshot), `render`/`hydrate` two-pass
    drivers, `NotaDoc`, smart punctuation, entities. Plus the **host seam** for frameworks that
    own the render loop (`route.ts`/`shell.ts`/`doc-pass.ts`): `notaRoute(Doc)` runs
    `collectDocState` and renders against that seed from *inside* the host's `renderToString`,
    parking the converged pass on a request-scoped channel for `NotaDocState` — placed after the
    app in the host's shell — to emit and convergence-check. None of it imports SolidStart.
  - **prelude** — the ambient stdlib: `Tex` (KaTeX→MathML), `CodeInline`/`CodeBlock` (sync
    shiki, armed parts→decorations), the reference family (`Heading`/`Toc`/`Ref`/`Label`,
    notes as anchors+refs, `Cite`/`Bibliography`, `Figure`/`Subfigure`/`Caption` as
    figure-kind anchors with derived ordinals, `Smallcaps`), the definition-tooltip system
    (per-doc banks; Floating UI places the open tooltip), and session-owned
    `lstset`/`mathset`/`secset`/`bibset` config.
  - **compiler** — sync shim over the in-process wasm reader, which **lives in this package**:
    `build.mjs` copies `oxc/target/js` → `src/generated/` (gitignored) and re-exports it raw as
    `@nota-lang/compiler/reader`. Also the single-sourced emit-surface constants
    (`CORE_RUNTIME_NAMES`, `FRAMEWORK_MODULES`/`FRAMEWORK_PACKAGES`, `DOC_EXPORT_NAME`,
    `LINE_CLASSIFIERS`) every other package derives its name-lists from. No subprocess backend,
    no separate wasm package.
  - **vite** — the `.nota` transform + `nota()` preset, plus `@nota-lang/vite/solid-start`
    (`notaStart()`: the SolidStart v2 preset, an optional peer — it composes `nota({solid:false})`
    with `solidStart({extensions:["nota"]})`, since exactly one vite-plugin-solid may claim
    `.nota`). That preset does NOT offer file-system routing over `.nota`: SolidStart's router
    parses route files as TSX to find their exports, so a `.nota` under `routeDir` is dropped.
    Owns the **one-`solid-js`-per-page invariant** via `resolve.dedupe` (`DEDUPED_PACKAGES` = framework set +
    `SOLID_JSX_DIST_PACKAGES`). A new package that ships Solid-compiled JSX in its dist (like
    the retired paper/explorable) MUST join `SOLID_JSX_DIST_PACKAGES` — currently empty — or a
    host's derived `noExternal`/`optimizeDeps` lists silently miss it.
    It owns the **host's** singletons too (`dedupedPackages()`): every `@codemirror/*`,
    `@lezer/*` and `style-mod` found in the host's own `node_modules`, discovered per project
    rather than listed, since deduping a name the root cannot resolve turns a working import
    into an unresolved one (Vite sets the basedir to the root and does not fall back). The
    scopes are taken whole — a `Language`, a parser and a `Tag` are all compared by reference,
    so there is no line to draw inside them. What dedupe cannot reach — a singleton nothing
    depends on *directly*, dragged in twice by two `lang-*` from different stores —
    `duplicateSingletons()` reports as a `buildEnd` warning naming both paths; the fix is for
    the host to make it a direct dependency. `@nota-lang/codemirror` peers the four it imports
    (`@codemirror/state|view|language`, `@lezer/highlight`) so the app owns that copy; a
    `link:`ed checkout still resolves from its own store, which is what the dedupe is for.
  - **cli** — `nota build doc.nota → doc/`: two programmatic vite builds (SSR render, then

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nota-lang/nota](https://github.com/nota-lang/nota) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
