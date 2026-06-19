---
trigger: always_on
description: You operate inside **Claude Design Web** under a document-backed premium UI protocol.
---

# Claude Design Premium Protocol

You operate inside **Claude Design Web** under a document-backed premium UI protocol.

Claude Design Web has a fixed, closed set of native product Skills. This starter does not install new
native Skills. Instead, this root `CLAUDE.md` acts as the bootstrap file, and the `.skill.md` files
are **documental procedures**: reusable operating instructions that you apply when relevant. Apply
only the procedures relevant to the current task. Do not run every procedure automatically.

## Core context

- `CLAUDE.md`: workflow, routing, and behavior rules (this file).
- `DESIGN.md`: visual identity, layout principles, and aesthetic constraints.
- `styles.css`: greenfield starter CSS facade for Claude Design's native compiler.
- `starter-kit/static/tokens.css`: greenfield starter token source for color, spacing, typography, radius, elevation, and motion in the canvas.
- Brownfield exports may instead use existing CSS entries such as `colors_and_type.css`; inspect `_ds_manifest.json.globalCssPaths` and preserve the existing token source.
- `design-tokens.json`: generated/reference token artifact for documentation and later handoff.
- `skills/*.skill.md`: document-backed procedures for design-system enforcement, audits, polish, implementation review, and final checks.
- `templates/<slug>/index.html`: optional native Claude Design template track for greenfield starter work.
- `components/*.jsx` + `components/*.d.ts`: optional native component examples exported for the compiler.
- `components/<Name>.html` or `preview/*.html`: native `@dsCard` specimens, depending on project shape.
- `starter-kit/static/`: optional static HTML/CSS/JS scaffold for the Claude Design Web canvas.
- `starter-kit/`: optional portable patterns for later Astro, Vite, or Next implementation.
- `CLAUDE-DESIGN-SEED.md`: optional one-shot bootstrap seed for an empty Claude Design Web canvas.

## Validation canary

Use this exact token only when the user is validating whether the root file loaded:

```text
CDP-CLAUDE-OK
```

Do not add it to every normal first response. In projects where root loading is already proven, the
canary is a diagnostic, not a ritual.

## Native Claude Design hooks

Use native Claude Design hooks before inventing replacements, while preserving the shape of the
current project:

- Tokens: use the existing CSS token source. In this starter, that is root `styles.css` importing
  `starter-kit/static/tokens.css`; in brownfield exports, it is often `colors_and_type.css` or the CSS
  files listed in `_ds_manifest.json.globalCssPaths`.
- Templates: when using the greenfield template track, prefer `templates/<slug>/index.html` with
  `<!-- @template name="..." description="..." -->` near the top.
- Specimen cards: use `@dsCard`. Greenfield component examples can colocate sidecars such as
  `components/Botao.html`; brownfield exports often use `preview/*.html` and `ui_kits/*/index.html`.
- Components: when using namespace components, use `.jsx` + `.d.ts` files with named exports and let
  the compiler expose the generated namespace. Do not assume every export uses this path.
- Live proof: run Claude Design's native `check_design_system` when available.

Repo-side scripts are maintenance preflight only. They do not replace the native in-session check.

## Canvas runtime constraints

Claude Design Web is a static authoring and preview canvas for this workflow. Do not try to run git,
package installs, package scripts, lint, tests, Vite, Next, Astro, or any dev server inside the
canvas. Use direct HTML/CSS/browser JS first. Treat Astro, Vite, and Next as later handoff targets.
Do not use `starter-kit/index.js`, ESM imports/exports, npm packages, or bundler-dependent modules
inside the canvas. Self-contained UMD/IIFE globals may be loaded with `<script src>` if they were
built outside the canvas and expose their API on `window`.

React/JSX is allowed only as an in-browser escape hatch: fixed React/ReactDOM UMD script tags +
Babel standalone + `<script type="text/babel">`. Do not use npm imports or bundler-style module
graphs in the canvas. For portable starter examples, expose reusable components on `window`; in
observed Claude Design UI kits, Babel scripts may also share globals within the page. Prefer vanilla
JS unless real component state or interaction justifies React.

The `scripts/` directory contains local repo-side validation helpers. They are useful before handoff
or when maintaining this starter, but they do not run inside Claude Design Web.

When starting from an empty canvas, write the root `CLAUDE.md` first, then scaffold static files, then
ask opening questions before generating visual work. If a readable source starter folder is available,
replicate its real files instead of recreating code from memory.

## Token truth

- In greenfield projects created from this starter, `starter-kit/static/tokens.css` is the initial
  active source of token values.
- In brownfield Claude Design exports, do not impose `styles.css` as the token source. Use the
  existing CSS token file, commonly `colors_and_type.css`, and verify it against
  `_ds_manifest.json.globalCssPaths`.
- Static canvas pages must use CSS custom properties from the active token CSS file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oalanicolas/claude-design-premium](https://github.com/oalanicolas/claude-design-premium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
