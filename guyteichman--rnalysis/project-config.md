---
trigger: always_on
description: Operational guide for Claude Code (and any AI/agent) working in this repository.
---

# CLAUDE.md — working in RNAlysis

Operational guide for Claude Code (and any AI/agent) working in this repository.
Read this first. Deeper material lives in [`.claude/context.md`](.claude/context.md),
[`.claude/workflows.md`](.claude/workflows.md), and [`.claude/design-philosophy.md`](.claude/design-philosophy.md).

> **Not using Claude Code?** This guide applies to you too. [`AGENTS.md`](AGENTS.md) is the
> provider-neutral entry point for any AI agent or automated contributor — it distills the
> non-negotiable rules and points back here for the full detail. `CLAUDE.md` (this file) is the
> canonical, complete version; keep the two in sync when you change the rules.

---

## What RNAlysis is

RNAlysis is desktop software that lets biologists analyze RNA-sequencing data —
from raw FASTQ through filtering, normalization, differential expression, clustering,
and enrichment — **without writing a single line of code**. It ships both a PyQt6
graphical app and a programmatic Python API, and it targets scientists with **zero
programming experience**. Correctness and reproducibility are non-negotiable
(see [design-philosophy](.claude/design-philosophy.md)).

---

## The one thing to understand first: the API↔GUI reflection contract

**The programmatic API is the source of truth. The GUI is generated from it by reflection.**
Internalize this before changing anything, because it explains most of the conventions.

When you add a public method to a `Filter`/`FeatureSet` subclass (or a public function in
`fastq`/`enrichment`), the GUI **automatically discovers and renders it** — you do not write
Qt code for it. Three things drive that rendering:

1. **Discovery** — `gui.py::get_all_actions()` calls `dir()` on the object and keeps every
   public (non-`_`) callable not listed in that widget's `EXCLUDED_FUNCS`. Methods are sorted
   into GUI tabs by **name heuristics**: contains `normalize` → *Normalize*; contains `filter`
   or `split` → *Filter*; membership in `SUMMARY_FUNCS`/`CLUSTERING_FUNCS`/`GENERAL_FUNCS` →
   those tabs; otherwise → *Visualize*.

2. **Parameter widgets** — `gui_widgets.py::param_to_widget()` maps each parameter's **type
   annotation** to a specific Qt widget. **Type annotations are load-bearing UI.** Examples:
   - `bool` → toggle switch · `int` → spin box · `PositiveInt`/`NonNegativeInt`/`NegativeInt` → range-bounded spin box
   - `Fraction` → 0–1 slider/spin · `Color` → color picker · `ColorList` → multi-color picker · `ColorMap` → colormap combo
   - `ColumnName`/`ColumnNames`/`GroupedColumns` → table-column pickers
   - `Literal['a','b']` → combo box · `Union[..., Literal[...]]` → combo box with an "other…" field
   - `Union[T, None]` (i.e. `Optional[T]`) → a checkbox that enables/disables a `T` widget
   - `Sequence[str]` → gene-set combo box
   These custom types live in [`rnalysis/utils/param_typing.py`](rnalysis/utils/param_typing.py).
   **Choosing the right annotation is how you design the GUI.**

3. **Labels & help text** — the `@readable_name('Human readable label')` decorator
   (`utils/generic.py`) sets the button/window title; the reStructuredText `:param x: ...`
   docstring lines become the per-parameter help/tooltips. A public method **without**
   `@readable_name` still appears (using its function name), so hide internals with a leading
   underscore or an `EXCLUDED_FUNCS`/`EXCLUDED_PARAMS` entry.

**Consequence:** a rename, a signature change, or a changed/loosened type annotation on a
public API method silently changes the GUI. Treat any such change as *risky* (see rules below).
And because you can't *see* that silent change in a code diff, **any change that is visible in a
GUI dialog must ship with a screenshot on the PR** (rule 9) — rebuild the dialog straight from the
API with `packaging/capture_gui_dialog.py` (the `gui-screenshots` skill wraps the full workflow).

---

## Repository map

```
rnalysis/
  __init__.py            # version, settings keys, FROZEN_ENV flag
  general.py             # misc top-level helpers (parsing IDs, saving tables, settings paths)
  filtering.py           # PUBLIC API — Filter, CountFilter, DESeqFilter, FoldChangeFilter, Pipeline
  enrichment.py          # PUBLIC API — FeatureSet, RankedSet (GO/KEGG/user enrichment, set ops)
  fastq.py               # PUBLIC API — adapter trimming, alignment, counting pipelines (wrap CLI tools)
  utils/                 # IMPLEMENTATION (not user-facing)
    io.py                # web services (async aiohttp + rate-limit + cache), subprocess, R runner
    enrichment_runner.py # enrichment statistics engines
    clustering.py        # clustering algorithms
    differential_expression.py  # builds/executes R scripts for DESeq2 & limma-voom
    feature_counting.py  # featureCounts (Rsubread) bridge
    genome_annotation.py # GTF/GFF parsing & gene-length logic
    ontology.py          # GO/KEGG DAG handling
    param_typing.py      # custom annotation types that drive GUI widgets  ← load-bearing
    generic.py           # readable_name, GenericPipeline, parallel helpers, misc
    parsing.py, validation.py, settings.py, installs.py
  gui/                   # PyQt6 app — reflects the API; you rarely add per-function code here

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GuyTeichman/RNAlysis](https://github.com/GuyTeichman/RNAlysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
