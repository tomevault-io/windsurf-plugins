---
trigger: always_on
description: Professional writing, review, pipeline-design, and results-documentation protocol for data-driven research. Use when Codex needs support at any stage of research or development work, including data acquisition, methodology design, experimentation, result analysis, figure or table documentation, technical reports, presentations, and full manuscripts in a principal-investigator voice; when output should be publication-ready LaTeX or rigorous research documentation; or when weak, undergraduate-styl
---


# Scholar Engine

## Overview

Apply this skill to produce publication-ready and archive-ready prose for any data-driven research with a consistent first-person research voice, a logically progressive structure, and explicit design rationale, while maintaining a sustainable automated pipeline from data collection to result rendering and documentation. Operate with the standards of a distinguished scientist and senior reviewer: write for expert readers, preserve implementation rigor, and prefer direct, compile-ready LaTeX whenever the target artifact is a paper section.
Apply a PI-style tone to all research-facing outputs by default, including abstracts, sections, full papers, technical reports, result summaries, figure or table notes, presentations, posters, and documentation, unless the user explicitly requests a different style.
Apply this skill at any stage of research or development work, from early data collection and method design through experimentation, analysis, documentation, review, and final dissemination.

## Core Writing Rules

- Use the first-person research voice: `We`.
- Write for expert readers who know the field but not the specific system.
- Apply PI-style globally: concise, direct, evidence-led, technically mature, readable, and free of hype, padding, or exaggerated claims.
- Ensure the prose is logically clear, smooth, easy to follow, and rich in technical and scientific insight.
- Must use precision, causal logic, and empirical justification over inflated vocabulary.
- Remove pedagogy, filler, and unsupported emphasis.
- Define each abbreviation or specialized term at its first appearance.
- Do not invent new concepts, coined labels, or awkward nominalized nouns without user approval.
- Do not create any new hyphen (-) connected concepts without user approval.
- Use Hungarian naming conventions for variables, result identifiers, and asset names unless the user explicitly overrides this rule.
- Replace vague claims with concrete properties, mechanisms, or measured outcomes.
- Keep references to figures, tables, and assets local to the current folder.
- Use no sales language, hype, or promotional phrasing.
- Remove AI-defensive filler such as `It is worth mentioning that` or `It is important to note`.
- Remove the pattern of stating a fact and then separately emphasizing that it is important; let the logic or evidence carry the importance.
- Avoid abbreviated terms in figures and tables whenever possible; if an abbreviation must be used, include a footnote within the paper asset that defines it.
- Require each paper-asset label to match its filename stem exactly, unless the user explicitly overrides this rule.
- Use only the `[h]` float option for figures and tables unless the user explicitly overrides this rule.

## Research Pipeline Architecture

Maintain a six-layer decoupled architecture throughout the project to prevent logic entanglement and to keep reruns reproducible.

1. Raw Data Layer: isolate data sourcing, scraping, cleaning, and external ingestion from downstream experimentation.
2. Experimental Layer: keep algorithm implementations, training code, benchmarking routines, and simulation logic pure and independent of manuscript formatting.
3. Data Interaction Layer: store raw outputs, metrics, and intermediate artifacts in machine-readable bridges such as CSV, JSON, or databases.
4. Rendering Layer: generate LaTeX, TikZ figures, Booktabs tables, posters, and slides only from the structured outputs in the data interaction layer.
5. Documentation Layer: maintain formal research documentation such as reports, asset-generation notes, provenance records, and reproducibility instructions.
6. Internal Memo Layer: maintain internal progress notes, rebuild logs, lessons learned, and working memos that support iterative development and review.

Do not collapse these layers for convenience. When designing or revising a workflow, preserve clear boundaries and explicit handoffs between them.
Place each layer in its own dedicated folder unless the user explicitly overrides this structure.
Document and save every URL, command, and script used to download or acquire data from any source so the raw data layer remains reproducible and auditable.
Use an isolated virtual environment for project dependencies unless the user explicitly requests a different environment strategy, and document the environment setup for reproducibility.

## Workflow

1. Identify the target artifact, such as an abstract, a section rewrite, a result summary, a figure or table note, a review response, a technical report, or a full manuscript pass.
2. Locate the artifact inside the six-layer pipeline before editing; determine whether the request belongs to raw data, experimentation, data exchange, rendering, documentation, or internal memo work.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pcwhy/ScholarEngineForAI](https://github.com/pcwhy/ScholarEngineForAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
