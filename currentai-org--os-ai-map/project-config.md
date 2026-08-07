---
trigger: always_on
description: This file provides guidance to coding agents and assistants when working in this repository.
---

# AGENTS.md

This file provides guidance to coding agents and assistants when working in this repository.

## Project Overview

`os-ai-map` is the public data + modeling home behind the AI Stack Map. It holds curated
YAML (`sources/`), warehouse SQL and fetchers (`warehouse/`), a deterministic build
pipeline (`build/`), and the published notebook (`notebooks/`).

There is no front-end in this repo. The website lives in the `aipotluck.org` monorepo
(`currentai-org/aipotluck.org`), which *consumes* this data and does not regenerate it.
The older `os-ai-visualization` repo is retired; it still receives bot data-sync PRs, so
activity there is not a signal of real work.

## Directory map

```
sources/               Curated YAML: organizations, categories, products, scores
sources/taxonomy.yaml  Arc grouping + cross-category display order
sources/signal_routing.yaml  Which machine signal is authoritative per dimension, and
                       which values mean "this source has no answer" (abstain_values)
sources/evidence_policy.yaml  When an observation is admissible as evidence
sources/rubrics/       Shared scoring ladders. A category inherits one with
                       `scoring_recipe: {extends: <name>}` rather than copying it;
                       or, for a category whose products don't all climb the same
                       ladder, `{extends: {<product type>: <name>, ...}}` (safeguards
                       is the example). build/rubrics.py resolves either form.
                       license-to-tier lives here, because whether AGPL is `osi` is
                       a fact about AGPL, not about one category.
warehouse/models/      UDM SQL (entities, events, metrics, scores)
warehouse/ingest/      Python fetchers that write CSVs to warehouse/catalog/
warehouse/catalog/     Raw external CSVs (HF benchmarks, incidents, GitHub orgs)
warehouse/sources.yaml Manifest: each external source declares EITHER a fetcher
                       (writes a CSV) or an ingested_by (a UDM reads it directly)
build/                 Python pipeline, see below
notebooks/             Generated ai-stack-map.py and standalone companion notebooks (pypi-geo-trends, oss-ai-trends, long-tail-explorer)
docs/methodology.md    Canonical methodology copy, rendered into the notebook (a build input)
docs/guides/           Query conventions, notebook style, freshness and verification
docs/runbooks/         Maintainer deploy runbooks
docs/schemas/          JSON Schemas for the source files (four concerns + taxonomy)
skills/                Agent skills for common editor workflows
tests/                 pytest suite for build helpers and serializer behavior
```

### What is in `build/`

Every module is a CLI with a docstring that explains why it exists; run any of them with
`--help`. Grouped by what they are for:

```
Notebook build      validate.py      sources/ schema + cross-file invariants
                    serialize.py     sources/ -> build/notebook_data.json
                    render.py        notebook_data.json -> notebooks/ai-stack-map.py
                    update_readme.py syncs the README stat badges
                    slugs.py         slug helpers shared by the above

Config bridge out   serialize_registry.py  identity: what exists
                    serialize_rubric.py    each category's rubric + recorded evidence
                    publish_registry.py    pushes both table sets to OSO as static models

Scores back in      apply_scores.py   reads computed scores from OSO, writes
                                      openness.score and openness.class into
                                      sources/scores/ and nothing else. The ONLY
                                      inbound data path. It writes no dates -- see
                                      docs/guides/verification.md.

Checkers (CI)       check_rubric.py    does the rubric reproduce the hand-authored scores
                    check_routing.py   which dimensions have a usable machine signal
                    check_freshness.py how stale is each axis

Proposers           propose_arxiv.py     candidate arXiv ids, verified live
                    propose_artifacts.py candidate artifacts, verified live
```

Proposers deliberately **print rather than write**. Matching artifacts by name measured 2
correct in 10 on this data, and a wrong artifact attaches another project's license and
downloads to a product, which is indistinguishable from a real score until someone checks.

## Data model

The curated source set is four per-record YAML concerns in `sources/` plus the single
`sources/taxonomy.yaml` manifest:

- **organizations**: one file per org (`name`=slug, `display_name`, `type`, `homepage`,
  optional `github` typed-url array and `comments` string). Owns the `products:` roster: a list of product slugs that belong to this org. A product
  slug must appear in exactly one org roster (validated).
- **categories**: one file per stack-map category (`name`=slug, `display_name`). Owns the
  ordered product roster (`products:` array). Order equals display order. One product
  appears in exactly one category. Category files no longer carry `arc` or cross-category
  `order`. Optional `comments` string for curator notes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [currentai-org/os-ai-map](https://github.com/currentai-org/os-ai-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
