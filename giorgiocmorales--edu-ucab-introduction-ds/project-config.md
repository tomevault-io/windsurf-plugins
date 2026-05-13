---
trigger: always_on
description: This repository is the instructor production workspace for UCAB's Introduction
---

# AGENTS.md

## Purpose

This repository is the instructor production workspace for UCAB's Introduction
to Data Science course. Treat it as a course-authoring repository, not as a
package or general website.

## Priorities

1. Reproducibility
2. Course consistency
3. Conservative editing
4. Clear structure
5. Renderable outputs

## Source of Truth

- Lecture source lives in `lectures/`.
- Shared course-wide lecture styling lives in `lectures/_format/`.
- Shared lecture metadata lives in `lectures/_metadata.yml`.
- Lecture starter files live in `lectures/_templates/`.
- Problem-set source and templates live in `problem-sets/`.
- Canvas-ready and instructor-only generated assessment outputs belong under
  `outputs/problem-sets/` and should not be committed.
- Automation scripts live in `scripts/`.
- Plans live in `plans/`.
- Repo-local skills live in `.agents/skills/`.

When a task matches a repo-local skill, read that skill before editing.

## Folder Rules

Each lecture folder should normally contain only these root-level entries:

- `slides.qmd`
- `handouts/`
- `assets/`

Lecture-specific images and files belong in that lecture's `assets/` folder.
Reusable logos, backgrounds, CSS, and shared format logic belong in
`lectures/_format/`.

Do not keep rendered outputs, `slides_files/`, ad hoc exports, copied render
artifacts, or cache folders inside lecture source folders. Generated artifacts
belong under `outputs/`.

`data/`, `references/`, and `legacy/` are source or archival areas. They may be
read as source material, but do not reorganize, move, or delete their contents
unless explicitly asked.

## Lecture Format Rules

Use the shared format in `lectures/_format/` for course-wide styling,
backgrounds, logos, and Reveal behavior. Individual lectures should not
introduce local styling overrides unless explicitly revising the shared format.

Every completed lecture should start and end with:

```markdown
## Main Message {.main-message}
```

Use `#` headings for major lecture sections and `##` headings for individual
slides.

Use `## Intervention Space {.intervention-slide}` for in-class checks. Prefer
including both a prompt and a hidden answer revealed with `::: fragment`.

Lecture 01 is intentionally broad because it introduces course logistics,
R/RStudio, and first code. Future lectures should normally be more focused.

## Lecture Authoring Conventions

- Lecture slides should follow the shared Quarto/Reveal structure used in this
  repo.
- Explain each new student-facing R function, operator, or workflow the first
  time it is taught.
- R code shown in lecture slides should normally be presented incrementally as
  the lecture progresses, rather than dumped all at once.
- When feasible, code examples should be broken into pedagogically meaningful
  fragments that reveal step by step.
- Introduce the concept in prose before showing code, and add interpretation
  text after code when students should notice a result or consequence.
- Code comments inside visible chunks should label conceptual steps.
- Avoid long uninterrupted code blocks unless the teaching goal requires showing
  the full script at once.
- Use hidden setup chunks with `#| include: false` only for package/data setup
  needed for rendering but not taught on that slide.
- Lecture images should include `fig-alt` text where practical.

## Handout Rules

Handout scripts should be an exact runnable mirror of slide examples.

Handout scripts should:

- follow the lecture flow
- preserve continuous course-wide numbering across the full course
- begin with comments identifying the lecture number and handout title
- use RStudio-style section comments, such as
  `# PACKAGES ---------------------------------------------------------------`
- stay runnable as standalone teaching scripts

If a lecture introduces a new package required for rendering or handouts, update
`scripts/setup_project.R`.

## Problem Sets and Assessments

Problem-set source lives in `problem-sets/`.

Use separate artifact labels:

- script problem set: `.R` assignment scaffold
- rendered problem set: `.qmd` assignment scaffold
- Canvas-ready question bank: instructor-facing assessment content with answer
  keys and validation code

Problem sets should be consolidated in a format that can be copied into Canvas.
Elaborate coding problems should also have corresponding R scripts for
instructor review.

Canvas-ready and instructor-only generated outputs should live under
`outputs/problem-sets/` and should not be committed. GitHub should track source
files and reproducible scripts, not final Canvas exports or shareable rendered
artifacts.

## Source Material Policy

When creating or revising lectures, treat `legacy`, existing lecture materials,
and `references` as important source material.

For first lecture drafts, relevant legacy scripts define the minimum procedural
coverage. Adapt examples to the current course arc, data, and scaffold as
needed, but every substantive procedure from the relevant legacy scripts should
be reflected in essence unless the omission is documented.

Preserve source lineage and avoid drifting away from established course content
without a clear reason.

External image sources, textbook references, and adapted legacy-material notes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giorgiocmorales/edu-ucab-introduction-ds](https://github.com/giorgiocmorales/edu-ucab-introduction-ds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
