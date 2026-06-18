---
trigger: always_on
description: This file provides guidance to Codex when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex when working with code in this repository.

<!-- ============================================================
     TEMPLATE FILE — customize every section marked CUSTOMIZE
     See SETUP.md for a step-by-step checklist.
     ============================================================ -->

## What This Is

<!-- CUSTOMIZE: Replace with a description of your paper -->
This repository contains the LaTeX source for "[Paper Title]". A research paper on [topic]. The paper targets [journal/conference format].

The goal is to produce a technically rigorous, internally consistent, publication-ready scientific manuscript. Edits must improve clarity, precision, coherence, and argument strength without changing the intended technical meaning.

**Research questions** (anchor for consistency checks):
<!-- CUSTOMIZE: Replace with your actual research questions -->
- **RQ1:** [Your first research question]
- **RQ2:** [Your second research question]

## Writing Objectives

Prioritize, in this order:

1. Technical correctness
2. Consistency with the rest of the manuscript
3. Clarity and precision
4. Concise academic style
5. Grammar and fluency

Do not make stylistic changes that weaken technical precision.

## Writing Style

- Use formal scientific English.
- Prefer precise terminology over stylistic variation.
- Prefer direct, declarative sentences.
- When introducing a tool or system, prefer "This work uses X" over "X serves as Y" — keep the authors as the active agent.
- When introducing a contribution, prefer "This work proposes X to achieve Y" — make clear it is a contribution of this paper.
- Avoid conversational wording, promotional tone, and vague intensifiers.
- Avoid vague relative clauses that restate a role without adding information.
- Avoid redundancy across adjacent sentences and paragraphs.
- Avoid introducing new terminology when existing terminology already covers the concept.
- Preserve the manuscript's established terminology unless there is a strong reason to change it.
- Preserve the intended meaning of claims, definitions, assumptions, and results.

## Terminology and Consistency

Before changing terminology:

- search the repository for the same term, acronym, and concept;
- check whether the term is defined elsewhere;
- keep terminology aligned across abstract, introduction, method, experiments, discussion, and conclusion.

When revising text:

- maintain consistent names for methods, modules, datasets, baselines, and metrics;
- maintain consistent capitalization of technical terms and acronyms;
- ensure acronym expansion is consistent with first use;
- ensure the contribution statements remain aligned with the body text.

<!-- CUSTOMIZE: Add any project-specific terminology rules here. -->

If a local edit would create inconsistency with the rest of the paper, flag it explicitly.

## Technical Content Rules

- Do not add technical claims, assumptions, or results unless supported by the manuscript context.
- Do not silently strengthen claims.
- Do not silently weaken limitations.
- Do not introduce citations not already present in the bibliography unless explicitly asked.
- Do not alter equations, symbols, labels, or references unless necessary.
- Preserve all LaTeX labels, refs, cites, and macros.
- Preserve comments unless they are obsolete or clearly misleading.

## Structure-Aware Editing

When editing a section, verify alignment with:
- title and subsection purpose,
- previous definitions,
- method terminology,
- experiment setup,
- stated contributions,
- conclusion wording.

When rewriting:
- preserve paragraph function;
- preserve the logical sequence;
- avoid moving content across sections unless explicitly asked.

## Preferred Editing Behavior

For small edits:
- make the minimal change that resolves the issue.

For larger rewrites:
- first infer the rhetorical role of the section;
- then rewrite to improve flow while preserving technical meaning.

For ambiguous passages:
- prefer a conservative rewrite;
- if ambiguity cannot be resolved locally, leave a short note explaining the issue.

## LaTeX Handling

- Keep LaTeX syntax valid.
- Do not break environments, labels, references, or bibliography commands.
- Do not replace structured LaTeX constructs with plain text.
- Keep math notation intact unless explicitly asked to revise it.
- Do not alter formatting macros unless necessary for correctness or consistency.

## Validation Workflow

Before editing:
- inspect the target file;
- search for related terminology, definitions, and claims in the repository.

After editing:
- verify that references, citations, and labels are intact;
- check that the revised text matches surrounding style and terminology;
- compile the manuscript if a build command is available;
- report any warnings or errors introduced by the change.

## What to Report Back

When making substantive edits, report:
- what was changed,
- why it was changed,
- any terminology or consistency issues discovered elsewhere,
- any follow-up sections that may need alignment.

## Non-Goals

Do not:
- perform broad stylistic normalization across the repository unless explicitly asked;
- paraphrase purely for variation;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rezenders/scientific_writing_agent](https://github.com/Rezenders/scientific_writing_agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
