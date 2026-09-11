---
trigger: always_on
description: Orientation for an AI agent working in this repository. Everything below was read
---

# AGENTS.md — sparrowhawk

Orientation for an AI agent working in this repository. Everything below was read
from the tree on `main`; where something could not be verified it says so.

For *how to add a new tool*, read [SKILLS.md](SKILLS.md) — it is self-contained
and needs neither this file nor the guide. [`docs/src/wasm_guide.md`](docs/src/wasm_guide.md)
is the same WebAssembly material written for a human reader.

---

## What this is

A browser-based bioinformatics platform. Rust crates compile to WebAssembly and
run **entirely client-side** — no server, nothing uploaded. Each tool is a tab in
a single Vue 3 SPA.

The repository was renamed: this is `bacpop/sparrowhawk` (formerly
`sparrowhawk-web`), and the assembler crate is now `bacpop/sparrowhawk-asm`.
Older checkouts and documents may still use the old names.

Eight tools, one page each under `www/src/components/pages/`:

| Tool | Page | Crate |
|---|---|---|
| Assembly | `AssemblyPage.vue` | `sparrowhawk-asm` |
| Mapping / Alignment | `MappingAlignmentPage.vue` | `ska.rust` |
| Taxonomic ID | `TaxonomicIDPage.vue` | `sketchlib.rust` |
| Gene calling | `GeneCallingPage.vue` | `orphos-bridge` |
| Host depletion | `HostDepletionPage.vue` | `deacon-bridge` |
| AMR detection | `AMRDetectionPage.vue` | `sparrowhawk-amr` |
| Protein embeddings | `ProteinEmbeddingsPage.vue` | `esm-bridge` |
| Transmission | `TransmissionPage.vue` | (front-end analysis) |

Plus `FaqPage.vue`.

---

## Getting a working checkout

```sh
git clone --recurse-submodules https://github.com/bacpop/sparrowhawk.git
cd sparrowhawk/www && npm install && npm run serve
```

**Without `--recurse-submodules` the `rust/` submodule directories are empty and
every wasm build fails.** This is the most common way to arrive at a broken tree.
Recover with `git submodule update --init --recursive`.

Requires the Rust toolchain plus `wasm32-unknown-unknown`, and `wasm-pack`.

### Two facts that will otherwise cost you an hour

1. **There is no hot reload.** `www/vue.config.js` sets `hot: false`,
   `liveReload: false` and `watchFiles: { paths: [] }` deliberately — otherwise
   every file touch retriggers seven `wasm-pack` release builds. **Restart
   `npm run serve` to see any change**, front end included.
2. **First start is slow.** Seven crates compile in release mode before webpack
   runs.

---

## Layout

```
sparrowhawk/
├── docs/            mdBook: user docs + the WebAssembly developer guide
├── electron/        desktop shell
├── rust/            7 crates — 4 submodules, 3 in-tree
├── www/             the Vue 3 SPA
├── .github/workflows/   Cloudflare Pages deploys (app + docs)
└── netlify.toml     older deploy path, still present
```

### `rust/`

Four **git submodules**, per `.gitmodules`:

| Crate | Repo | Branch |
|---|---|---|
| `sparrowhawk-asm` | `bacpop/sparrowhawk-asm` | `master` |
| `ska.rust` | `bacpop/ska.rust` | `sparrowhawk-dev` |
| `sketchlib.rust` | `bacpop/sketchlib.rust` | `sparrowhawk-dev` |
| `sparrowhawk-amr` | `bacpop/sparrowhawk-amr` | `master` |

Three **in-tree bridge crates**, which wrap an upstream library for the browser:
`deacon-bridge`, `orphos-bridge`, `esm-bridge`. `esm-bridge` also carries
`build.rs`, `model/`, `scripts/` and `tests/`.

### `www/src/`

| Path | Contents |
|---|---|
| `assets/app.css` | the entire design system (see below) |
| `components/pages/` | one page per tool, plus `amr-detection/`, `protein-embeddings/`, `taxonomic-id/` subfolders |
| `components/ui/` | shadcn-vue primitives over `reka-ui` |
| `components/help/` | per-tool help collapsibles |
| `components/` | result displays, `MSAViewer/`, `SequenceViewer/`, `MinimisedSequenceViewer/`, `gene-calling/`, transmission views |
| `lib/utils.ts` | `cn()` — `twMerge(clsx(...))` |
| `platform/` | `files.ts`, `gpu.ts`, `electron.d.ts` — web-vs-Electron abstraction and WebGPU detection |
| `store/` | Vuex: `index.ts`, `state.ts`, `actions.ts`, `mutations.ts`, `getters.ts` |
| `workers/` | driver + `*.worker.ts` pair per tool |
| `pkg*/` | wasm-pack output, generated, not committed |

---

## The design system

All of it lives in `www/src/assets/app.css`:

```css
@import "tailwindcss";
@import "tw-animate-css";
@import "@fontsource/dm-sans/400.css";   /* …500, 600, 700 */

@custom-variant dark (&:is(.dark *));

:root {
    --background: oklch(1 0 0);
    /* …the full shadcn token set, plus --sidebar-* … */
    --radius: 0.625rem;
}
.dark { /* … */ }
```

**Tailwind 4**, configured through `postcss.config.js`
(`@tailwindcss/postcss` + `autoprefixer`). `tailwind.config.js` exists but is
effectively vestigial — Tailwind 4 does not need its `content` array.

`www/src/components/ui/` holds twelve primitives: `button`, `collapsible`,
`dialog`, `input`, `separator`, `sheet`, `sidebar`, `skeleton`, `slider`,
`table`, `tabs`, `tooltip`.

Note there are **two UI libraries** in `package.json`: `reka-ui` (which the
`ui/` primitives wrap) and `primevue` + `tailwindcss-primeui`. `react` and
`react-dom` are present only because `taxonium-component` needs them.

### The page layout every tool follows

From `TaxonomicIDPage.vue` — copy this shape rather than inventing one:

```html
<div class="flex flex-col gap-6 md:flex-row md:gap-0">
  <div class="w-full md:w-[350px] md:shrink-0">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bacpop/sparrowhawk](https://github.com/bacpop/sparrowhawk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
