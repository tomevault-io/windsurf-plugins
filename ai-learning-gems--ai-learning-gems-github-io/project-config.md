---
trigger: always_on
description: Shared Quarto formatting: folder structure, index files, heading levels, LaTeX, includes
---


# Quarto Conventions

Shared Quarto formatting and structure rules for all textbook chapter workflows.

---

## Folder-Based Structure

Because IDE agents rewrite files from scratch on each edit, a single-file approach causes the entire chapter to be rewritten when editing any section. Instead, use a **folder-based structure** where:
- Each section is a separate file (can be edited independently)
- An index file includes all sections (renders as a single document)

### File Structure

```
Statistics/
├── Bayesian Credible Intervals.qmd          ← Index file (includes all sections)
└── Bayesian Credible Intervals/             ← Folder (same name as index)
    ├── TEXTBOOK-PLAN.md                     ← Created by the research workflow
    ├── _01-introduction.qmd                 ← Section 1
    ├── _02-the-bayesian-framework.qmd       ← Section 2
    ├── _03-computing-credible-intervals.qmd ← Section 3
    ├── _04-examples.qmd                     ← Section 4
    ├── _98-math-background.qmd              ← Math Background appendix (if needed)
    ├── _99-closing.qmd                      ← Summary, questions, resources
    └── sources/                             ← Symlink or note pointing to AI-Learning-Gems/sources/
```

**Key conventions:**
- **Index file:** `[Topic Name].qmd` — contains YAML header and `{{< include >}}` statements
- **Folder:** `[Topic Name]/` — same name as the index file (without `.qmd`)
- **Section files:** Prefixed with `_` so Quarto doesn't render them standalone
- **Numeric prefixes:** `_01-`, `_02-`, etc. for ordering (starting at `_01`, not `_00`)
- **Sources:** Referenced from central `AI-Learning-Gems/sources/` — see TEXTBOOK-PLAN.md for paths

---

## Index File Template

The index file should contain ONLY the YAML header and include statements. Most settings (format, jupyter kernel, execute options) are inherited from the project's `_quarto.yml` file — the index file only needs the title and the D2 diagram filter.

**CRITICAL:** The filter path is RELATIVE from the `.qmd` file to the `_extensions` folder at the project root. Adjust the number of `../` based on folder depth:
- Files 1 level deep (e.g., `Statistics/Topic.qmd`): `../_extensions/...`
- Files 2 levels deep (e.g., `Statistics/Subfolder/Topic.qmd`): `../../_extensions/...`

Example:

```yaml
---
title: "Bayesian Credible Intervals"
filters:
  - ../_extensions/pandoc-ext/diagram/diagram.lua
---
```

```markdown
{{< include "Bayesian Credible Intervals/_01-introduction.qmd" >}}

{{< include "Bayesian Credible Intervals/_02-the-bayesian-framework.qmd" >}}

{{< include "Bayesian Credible Intervals/_03-computing-credible-intervals.qmd" >}}

{{< include "Bayesian Credible Intervals/_04-examples.qmd" >}}

{{< include "Bayesian Credible Intervals/_99-closing.qmd" >}}
```

**CRITICAL:** The `{{< include >}}` shortcode must be:
- On its own line
- With blank lines above and below
- Path is relative to the index file location
- **ALWAYS quote the path** (wrap in `"..."`) — Quarto splits on spaces, so unquoted paths like `Vision Transformers/file.qmd` will fail with "could not find file" errors

---

## Section File Template

Each section file should NOT have a YAML header (it inherits from the index file). Start directly with the section heading:

```markdown
## Introduction {#sec-introduction}

Content goes here...

### Subsection A

### Subsection B
```

**CRITICAL — heading levels:** Each section file must have exactly **ONE `##` heading** (the section's main heading). All sub-parts within the file must use `###` or lower (`####`, etc.). This is because Quarto counts all `##` headings sequentially across `{{< include >}}`'d files — if a section file has 5 `##` headings, it consumes 5 section numbers, throwing off the numbering for every subsequent section.

**Do NOT use `.unnumbered` on any section heading.** All sections (introduction, body, closing) should be numbered. The `number-sections: true` setting in `_quarto.yml` handles this globally. Using `.unnumbered` causes subsections to render as 0.1, 0.2, etc., which looks broken.

---

## Per-Section Source Headers (Collapsible)

**Each section should include its own sources** as a collapsible header at the top. This keeps sources close to the content they support.

**CRITICAL: The section heading (`##`) MUST come FIRST, then the sources callout inside it.** If the callout is placed *above* the heading, it renders as belonging to the previous section.

```markdown
## Section Title {#sec-section-name}

::: {.callout-note collapse="true" title="Sources for this section"}

| # | Source | Summary |
|---|---|---|
| 1 | [ViT Paper](https://arxiv.org/abs/2010.11929) | Original ViT equations |
| 2 | [D2L ViT](https://d2l.ai/chapter_attention/vision-transformer.html) | Implementation details |

:::

[Section content...]
```

**CRITICAL — Attribution for Blog Content:**

Many sources in this project come from independent researchers' blogs (see `high-quality-blogs.md`). These are original intellectual contributions that MUST be attributed:

- **Source header**: Always list blog posts in the section source table with the author's name: `[Lilian Weng — "Reward Hacking in RL"](URL)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AI-Learning-Gems/AI-Learning-Gems.github.io](https://github.com/AI-Learning-Gems/AI-Learning-Gems.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
