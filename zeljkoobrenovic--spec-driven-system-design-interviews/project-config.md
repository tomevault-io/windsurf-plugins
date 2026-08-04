---
trigger: always_on
description: generates the image under `assets/generated/comic/` and writes the
---

# CLAUDE.md

Notes for future Claude sessions working on this directory. See `PLAN.md` for
the product spec and dataset schema — this file covers code conventions,
where logic lives, and common pitfalls.

## Keeping the docs in sync

This repo has overlapping documentation that must not drift apart:

- **`CLAUDE.md` and `AGENTS.md` are byte-identical.** `AGENTS.md` is a verbatim
  mirror of `CLAUDE.md` (for agents that read `AGENTS.md`). After editing
  `CLAUDE.md`, mirror it: `cp CLAUDE.md AGENTS.md` (or edit both identically).
  Never let them diverge — a reviewer should be able to `diff CLAUDE.md
  AGENTS.md` and get no output.
- **`README.md` files are human-facing** and describe the same shared facts at
  a higher level: the root `README.md` (project overview, build/run, layout)
  plus per-directory `README.md`s in `_templates/`, `data/`, and `docs/`.
- **When you change a shared fact** — a build/run command, the directory
  layout, how datasets are added, a core convention — **update all the docs it
  appears in**: this file (then mirror to `AGENTS.md`), the relevant
  `README.md`(s), and `PLAN.md` if it's a schema change. The detailed
  conventions and pitfalls live here in `CLAUDE.md`/`AGENTS.md`; the READMEs
  link here rather than duplicating the depth.

## What this is

A static system-design interview explorer. No framework, one CDN dep
(Mermaid v10). The pages have no bundler — the only "build" is a copy step
(`build.py`) that assembles deployable sites from sources into `docs/`.

Two pages, sharing `styles.css`:

- **`index.html` (overview)** — a visual grid of all interviews, organized into
  categories. Driven by `overview.js`. Each card shows an icon and links to the
  explorer via `interview.html#<datasetId>`.
- **`interview.html` (explorer)** — the per-interview step-by-step walkthrough.
  Driven by `interview.js` (one IIFE). This is the original single-page app.

> **"Group" is overloaded — watch out.** A *dataset group* is a top-level
> directory under `data/` (e.g. `examples`, `book`) that builds into one
> independent site `docs/<group>/`. A *category* is the `groups[]` array inside
> a single site's `index.json` (e.g. "Fundamentals", "Media & Search") — the
> sections the overview renders. The JSON key is `groups` for historical
> reasons; in prose we call those **categories**.

### Source layout (what you edit)

| Path                                | Role                                                     |
|-------------------------------------|----------------------------------------------------------|
| `_templates/index.html`             | Overview-page DOM shell (shared by every group)          |
| `_templates/overview.js`            | Overview-page behavior (one IIFE)                        |
| `_templates/interview.html`         | Explorer DOM shell + Mermaid CDN tag (shared)            |
| `_templates/interview.js`           | Explorer behavior (one IIFE, no modules)                 |
| `_templates/node-types.json`        | Canonical node types + external rendering config         |
| `_templates/styles.css`             | All styling, both pages (shared)                         |
| `_templates/icons/system-design.png`| Fallback interview icon (shared)                         |
| `data/<group>/index.json`           | One site's manifest (`groups[]` of categories)           |
| `data/<group>/<id>/interview.json`  | One dataset per subdirectory                             |
| `data/<group>/<id>/icon.png`        | Optional per-interview icon (else falls back)            |
| `data/<group>/<id>/assets/`         | Optional generated icons/images linked from JSON         |
| `_scripts/generate_interview_assets.py` | Generates interview assets and writes JSON links     |

Datasets are organized into **groups** (each a directory under `data/`).
`data/examples/` is the canonical group of worked examples; `data/book/` is the
book group (a pattern catalog + flagship cases like `payment-system` and
`notification-system`, plus `BOOK-STRUCTURE.md` as a planning note). A group is
publishable once it has an `index.json` manifest.

### Build output (generated — never hand-edit)

`build.py` produces one independent, deployable site per publishable group:

| Path                                | Role                                                     |
|-------------------------------------|----------------------------------------------------------|
| `docs/<group>/`                     | Copy of the whole `_templates/` tree (pages, js, css, `node-types.json`, `icons/`) |
| `docs/<group>/data/index.json`      | Copy of `data/<group>/index.json`                        |
| `docs/<group>/data/<id>/...`        | Copy of each dataset subdir (incl. any `icon.png`)       |

`docs/` is **committed** (GitHub Pages deploys from the `/docs` folder). After
changing anything in `_templates/` or `data/`, re-run `build.py` and commit the
regenerated `docs/`.

**Generated AI images are huge; downscale them as a separate step before
deploying.** The full-resolution originals under
`data/<group>/<id>/assets/generated/` are ~1500–3000px tall, but the explorer
only ever shows them in a `.diagram-image` box capped at `max-height: 560px`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeljkoobrenovic/spec-driven-system-design-interviews](https://github.com/zeljkoobrenovic/spec-driven-system-design-interviews) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
