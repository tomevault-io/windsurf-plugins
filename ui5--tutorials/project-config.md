---
trigger: always_on
description: This repository ships five OpenUI5 tutorials rendered as a GitHub Pages site with a client-side JavaScript / TypeScript language toggle. Every tutorial step is a self-contained, runnable UI5 app under `packages/<tutorial>/steps/NN/`. This document tells an agent enough about the layout, the toolchain, and the authoring conventions to be useful without re-deriving everything from source.
---

# Repository Guide for AI Agents

This repository ships five OpenUI5 tutorials rendered as a GitHub Pages site with a client-side JavaScript / TypeScript language toggle. Every tutorial step is a self-contained, runnable UI5 app under `packages/<tutorial>/steps/NN/`. This document tells an agent enough about the layout, the toolchain, and the authoring conventions to be useful without re-deriving everything from source.

## Layout

- `packages/quickstart/` — 3 introductory steps.
- `packages/walkthrough/` — 38 steps; the canonical reference for content shape and toolchain.
- `packages/databinding/` — 15 steps on data binding (JSON & resource models, property/two-way/one-way/aggregation/element/expression binding, formatters, data types, validation).
- `packages/navigation/` — 17 steps on routing and navigation.
- `packages/odatav4/` — 11 steps on OData V4.
- `packages/<pkg>/README.md` — the tutorial's overview and step index.
- `packages/<pkg>/steps.json` — machine-readable index of the tutorial's steps (`n`, `id`, `title`, `description`, `previewUrl`, `zipTs`, `zipJs`). Regenerate with `node _/generate-steps-index.js` after adding, removing, or renaming steps.
- `packages/<pkg>/steps/NN/` — one runnable app per step. Contains `README.md`, `webapp/`, `package.json`, `ui5.yaml`, and (for TS steps) `tsconfig.json`.
- `assets/` — CSS and JS that power the site (language toggle, code-couple tab container, `.ts/.js` extension substitution). Ships as-is to `dist/`.
- `tools/builder/` — build orchestrator (`node tools/builder`) that produces `dist/` for GitHub Pages: transpiles TS steps to JS, packages ZIP downloads, renders READMEs.
- `tools/dev-server/` — dev server (`npm start`) that mirrors the built site with live reload.
- `_/` — **gitignored** folder for one-off migration and maintenance scripts. Add throwaway tooling here; do not check it in as permanent tooling.
- `_includes/head-custom.html` — Jekyll partial that injects `assets/css/custom.css` and `assets/js/custom.js` into every GitHub Pages README render.
- `dist/` — build output, gitignored.

## Running the code

The repo is an npm workspaces monorepo. Each `packages/*/steps/*` is a workspace.

```sh
# one-time
npm install

# build every tutorial step + assemble dist/
npm run build

# preview the rendered site with working local live-preview links
npm start          # http://localhost:1337/packages/walkthrough/

# run one step directly
npm start -w ui5.tutorial.walkthrough.step07
npm start -w ui5.tutorial.odatav4.step03
# etc.

# typecheck every TypeScript step
npm run typecheck
```

Rely on `npm start -w <workspace>` when you need to see a specific step in a browser — the root dev server is heavier and only needed for site-level checks.

## Tutorial namespace and package naming

All five tutorials share one convention:

- App namespace: `ui5.tutorial.<pkg>` (used in `manifest.json` `sap.app.id`, `Controller` names, XML `controllerName`, `index.html` resource-roots, and JSDoc `@namespace`).
- Per-step `package.json` name: `ui5.tutorial.<pkg>.stepNN` (zero-padded).
- `ui5.yaml` `metadata.name`: `ui5.tutorial.<pkg>`.
- tsconfig `paths`: `"ui5/tutorial/<pkg>/*": ["./webapp/*"]`.

If you touch any of these, keep the naming aligned across the file set.

## Step README authoring conventions

Step READMEs are markdown rendered through GitHub Pages + a small client-side runtime in [assets/js/custom.js](assets/js/custom.js). Three conventions matter:

1. **Language-specific prose blocks.** Wrap TS-only content in `<details class="ts-only" markdown="1">…</details>` (mirror for `js-only`). The runtime converts these to `<section>` elements and CSS hides the inactive one. This is for *block-level* prose only — for a paired inline TS/JS link (e.g. a download-solution link), use `<span>` instead (see point 5).
2. **Adjacent ts/js code fences.** Write the TypeScript snippet in a `` ```ts `` fence and the JavaScript snippet in a `` ```js `` fence directly below (only whitespace between them). The runtime wraps the pair in a tabbed container.
3. **First-line path comments.** Every ts/js fence pair whose content is real source code must start with a `// path/to/File.ts` comment on line 1 (mirror `.js`). This is what tells a human reader — and an AI indexing the corpus — which file the snippet belongs to. For XML fences use `<!-- path -->`; for INI/properties use `# path`.
4. **File extensions in prose and headings.** Write `.ts/.js` where the current language should substitute. The runtime renders it as `.ts` in TS mode or `.js` in JS mode. Do not use the older `.?s` token — it was retired.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UI5/tutorials](https://github.com/UI5/tutorials) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
