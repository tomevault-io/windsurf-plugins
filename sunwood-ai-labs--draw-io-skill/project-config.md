---
trigger: always_on
description: Create, edit, export, and review draw.io diagrams. Use for native .drawio XML generation, PNG/SVG/PDF export, SVG overlap, border-overlap, label-intrusion, label-rect, short-terminal, text-contrast, text-emphasis, and text-overflow linting, layout adjustment, and AWS icon usage.
---


# draw.io Diagram Skill

## 1. Purpose

Use this skill when an agent needs to:

- create a new draw.io diagram as native `.drawio` XML
- edit or refactor an existing `.drawio` file
- export diagrams to `png`, `svg`, `pdf`, or `jpg`
- check routed edges, box- or frame-border overlap, supported non-rect shape border overlap, box penetration, short arrowhead tails, label collisions, weak text contrast, flat dark-card text treatment, or text overflow
- build architecture diagrams with AWS icons

This skill intentionally combines:

- the native draw.io assistant workflow used by Claude Code style tools
- practical XML editing and layout rules from repository use
- repository-ready SVG linting that catches issues draw.io does not flag

### 1.1 Repository structure

The repository layout and bundled workflow pieces are summarized in the diagram below.

![draw-io-skill structure](./assets/draw-io-skill-structure.drawio.png)

The repository also ships:

- an English structure source and exports under `assets/draw-io-skill-structure.drawio*`
- an icon block showcase under `assets/draw-io-skill-structure-icons.drawio*` plus a Japanese-localized companion under `assets/draw-io-skill-structure-icons.ja.drawio*`
- a shape-focused lint review sample under `assets/draw-io-skill-structure-shapes.drawio` with exports at `assets/draw-io-skill-structure-shapes.drawio.png` and `assets/draw-io-skill-structure-shapes.drawio.svg`
- a Japanese-localized companion source and exports under `assets/draw-io-skill-structure.ja.drawio*`
- default visual templates and style tokens under `references/aesthetic-templates.md`
- an editable polished visual template sample under `assets/aesthetic-template-sample.drawio`
- three professional-grade sample templates under `assets/aesthetic-sample-executive-dashboard.drawio`, `assets/aesthetic-sample-ai-pipeline.drawio`, and `assets/aesthetic-sample-security-incident.drawio`
- three purpose-driven templates with distinct existence reasons under `assets/purpose-board-brief-template.drawio`, `assets/purpose-dependency-orbit-template.drawio`, and `assets/purpose-incident-timeline-template.drawio`
- three additional purpose-driven templates under `assets/purpose-hypothesis-helix-template.drawio`, `assets/purpose-feature-value-matrix-template.drawio`, and `assets/purpose-value-conversion-sheet-template.drawio`
- public showcase pages under `docs/guide/showcase.md` and `docs/ja/guide/showcase.md`
- fixture-based lint coverage under `fixtures/basic`, `fixtures/border-overlap`, `fixtures/large-frame-border-overlap`, `fixtures/shape-border-overlap`, `fixtures/label-rect-overlap`, `fixtures/text-cell-overflow`, `fixtures/text-contrast`, `fixtures/text-emphasis`, and `fixtures/shape-text-overflow`

### 1.2 Repository-local commands

When working inside this repository, these are the main maintenance commands:

```sh
npm install
npm run check
npm run verify
npm ci
npm run docs:build
npm run docs:dev
uv run python -m py_compile scripts/find_aws_icon.py
```

Use them this way:

- `npm run check`: script syntax plus fixture-based lint verification
- `npm run verify`: full repository signoff, including docs build
- `npm run docs:build`: one-shot docs build
- `npm run docs:dev`: interactive docs preview

If you need to attach the repo as a skill in a local assistant environment, the repository docs use these conventions:

- Codex on Windows: junction `C:\Users\YOUR_NAME\.codex\skills\draw-io -> D:\Prj\draw-io-skill`
- Claude Code: clone under `~/.claude/skills/drawio`

### 1.3 Repository QA model

The repository uses three QA layers:

1. syntax checks for the JavaScript tools
2. fixture-based lint verification
3. documentation build validation for the public-facing docs

That keeps the technical tooling and the user-facing docs aligned in CI.

## 2. Default Workflow

Follow this order unless the user asks for something narrower:

1. Create or update the native `.drawio` file first.
2. When creating a diagram from scratch and the user did not specify a visual style, start from `references/aesthetic-templates.md`; default to the polished technical light template instead of draw.io's plain box defaults.
3. Keep `.drawio` as the editable source of truth for repository work.
4. If the user asked for an exported artifact, export to `.drawio.png`, `.drawio.svg`, `.drawio.pdf`, or `.drawio.jpg`.
5. Before showing, attaching, or claiming a diagram is ready, export an SVG and run the lint script.
6. If the draw.io CLI is unavailable, do not silently skip linting. Create or locate the expected companion `*.drawio.svg` if possible, run the lint script, and explicitly report any reduced coverage such as `parsed 0 edges`; then run a manual or scripted geometry sanity check for arrows, labels, and box intrusions before surfacing the image.
7. Open or surface the final artifact requested by the user only after linting and visual verification.
8. Even when lint passes, visually verify the result.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sunwood-ai-labs/draw-io-skill](https://github.com/Sunwood-ai-labs/draw-io-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
