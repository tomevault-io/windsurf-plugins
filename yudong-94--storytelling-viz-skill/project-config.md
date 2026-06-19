---
trigger: always_on
description: Story-first Plotly workflow for turning a dataset into a polished interactive visualization with local review artifacts. Use this skill when the user wants a concise, intuitive chart with a clear takeaway, plus final `index.html` and `preview.html`, without website publishing work.
---


# Storytelling Viz

Use this skill when the task is only about choosing, building, and refining the visualization itself.

## Portability Note

`SKILL.md` is the canonical cross-tool instruction file for this skill.

- Keep the core workflow and output contract here so the skill stays usable in both Codex and Claude Code.
- Keep tool-specific metadata, discovery config, or UI-only behavior outside this file when possible.
- If the repo includes example artifacts, treat them as demonstrations of the current workflow, not as an alternative source of truth.

The goal is to produce one Plotly visualization that is:

- story-telling
- intuitive
- concise

The default output of this skill stops at reusable local artifacts:

- `index.html`
- `preview.html`
- 2-3 insight bullets

Do not take on website publishing, repo posting, PR creation, or markdown post editing in this skill. If the user wants the visualization inserted into a site or blog post, use a separate publishing skill.

## Skill Promise

Unless the user explicitly asks otherwise, optimize for:

- one chart, not a dashboard
- one main takeaway, not a bundle of loosely related points
- comprehension without hover
- minimal interaction that adds value
- concise copy and restrained styling

Keep the honesty checks strict. The chart should feel clear and editorial, not generic, but never at the cost of comparability or truthfulness.

## When to Use This Skill

Use this skill when the user:

- provides a dataset and wants chart recommendations
- wants an embeddable interactive visualization in `Plotly.js`
- wants help matching an established visual style
- wants iterative draft-and-feedback refinement
- wants local review artifacts before publishing anywhere

Do not use this skill when the main request is about updating a website repo, Jekyll post, or GitHub Pages embed.

## Fast Path

Follow this default sequence:

1. Identify the data source and inspect the dataset shape.
2. Check comparability and chart logic.
3. Write the one-sentence story the viewer should remember.
4. Recommend one default chart and one fallback.
5. Ask the user to confirm the recommended direction before building, even when the choice seems straightforward.
6. Build the first draft in `Plotly.js`.
7. Run rendered visual QA on desktop and, when needed, a narrower layout.
8. Deliver the local artifacts, insight bullets, and any material caveats.

Keep the user in the loop at chart selection and at least once during refinement.

## Non-Negotiable Tests

Before moving from one stage to the next, apply these tests:

- `Story test`: the main takeaway can be stated in one sentence
- `Intuition test`: a first-time viewer can understand the main claim quickly without hover
- `Concision test`: if an element can be removed without hurting meaning, remove it
- `Honesty test`: the framing still matches the actual data, scope, and caveats

If the visualization fails one of these tests, simplify the chart, the copy, or both.

## Dataset Review

Before proposing a chart:

- confirm the dataset source and source URL or citation
- identify row count, column names, and data types
- classify each quantitative field by meaning when relevant: cost, income, rate, stock, flow, share, index, or timepoint
- detect time, geography, ranking, composition, distribution, and relationship fields
- check whether the compared values are actually comparable across year, geography level, currency basis, inflation basis, and methodology
- check whether the data supports one strong message or only several weaker ones
- note obvious cleaning issues that affect chart design
- flag concept mismatches early, such as mixing costs with income or combining stock and flow metrics in one direct ranking
- if the source is an article rather than a direct downloadable table, verify which subsection, embedded table, or figure the chart is actually using
- check whether the article headline, prose callouts, and ranking table use the same numbers
- if article prose and table values disagree, choose one authoritative subsection and say so in visible copy

If the user has not provided a data source, stop and ask for it before finalizing the visualization. Do not fabricate, infer, or omit the source when the origin of the data is unknown.

Treat this as mandatory, not best-effort:

- do not recommend a final chart, write story framing, or build the visualization until the user has provided the data source or clearly identified the source file/table being used
- if the user shares only a raw file without source context, ask for the underlying publisher, organization, dataset name, URL, or other citation before proceeding beyond exploratory inspection
- if the source cannot be established, stop rather than producing a stakeholder-facing artifact with an unverified origin

If key fields are not comparable, do not quietly proceed with a strong claim. Instead explicitly do one of the following:

- narrow the scope to a valid subset
- reframe the chart as a snapshot with caveats

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yudong-94/storytelling-viz-skill](https://github.com/yudong-94/storytelling-viz-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
