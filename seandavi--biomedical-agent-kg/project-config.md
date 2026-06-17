---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **Biomedical Agent Knowledge Graph** pipeline — a generated catalog of LLM-based
biomedical/bioinformatics agent systems and the entities worth traversing between them
(papers, repos, benchmarks, orgs, domains, tool collections, databases). The pipeline is
the `agentkg` uv package (`src/agentkg/`); `SPEC.md` is the design source of truth (draft
v0.4) — read it before changing the data model, vocab, or pipeline. `list.md` is a small
hand-authored crawl input for iteration; the real corpus is the five SPEC §11 awesome-lists
fetched via `--sources` (`sources.py`). `sample_graph.json` is an older example output.

## Running

uv-managed (Python provisioned by uv, ≥3.11). Entry point `agentkg`:

```bash
uv sync                                # install deps + create .venv
uv run agentkg run                     # mock backend (fully offline) -> data/graph.json
uv run agentkg run -b vertex           # live Gemini extraction (needs ADC, see .env)
uv run agentkg run -b vertex -n 2      # first N agents only (-n bounds classify spend too)
uv run agentkg run -b vertex -p 5      # also draft prose profiles for first N agents
uv run agentkg run -b vertex --sources # crawl the five SPEC §11 lists (~634 entries)
uv run agentkg run --log-level DEBUG   # loguru level (or KG_LOG_LEVEL)
uv run agentkg config                  # print resolved settings
```

Config is layered `.env` > process env > defaults via pydantic-settings (`config.py`):
`KG_*` for app settings, standard `GOOGLE_*` for Vertex (shared with gcloud/ADC). Live
runs use Vertex AI on project `bioc-u24`; Gemini 3.x needs `GOOGLE_CLOUD_LOCATION=global`
(regional endpoints 404 — see the memory note). No test suite or linter yet.

`.cache/` holds the iteration token-saver — namespaces: `classify`, `facets`, `profiles`
(model output keyed by `model+prompt+payload`), `context`/`openalex`/`repo`/`listmd`
(fetched text keyed by url). Re-running an unchanged prompt costs 0 tokens; editing a
system prompt auto-busts its keys. Delete `.cache/` to force a full rebuild.

## Core invariant — fix the generator, never the record

**Everything is generated.** No hand-curated data layer, no per-record override file
(no `corrections.yml`). When output is wrong, you fix the *generator* — the extraction
prompt or `rules.yml` (deterministic, class-level transforms) — then regenerate.
Rules catch **classes, not instances**: `{proteome, proteomic} → proteomics` is allowed;
"Biomni is multi_omics" is not. A per-record patch is a bug, not a fix.

The only human-touched inputs are **extraction prompts** (`backends.FACET_SYSTEM_PROMPT`)
and **`rules.yml`** — currently stood in by `vocab.py`: `DOMAIN_ALIASES`, `guard_vocab()`
(canonicalize + drop OOV), and the class-level reconcile rules in `pipeline.build` (e.g.
`multi_agent` dominates `single_agent` when sources disagree).

## Architecture

Two overlaid graph components: the curated catalog (agents + entities) and the paper
`cites` citation graph (default-OFF overlay — it dominates layout if always on).
**Agents are the spine.** Node vs. attribute rule: you *traverse through* nodes and
*filter on* attributes (benchmark → node; license/stars/language → attribute). A shared
node (specific database, tool collection) exists only once it connects ≥2 agents.

Node types: `Agent`, `Paper`, `Repo`, `Benchmark`, `Org`, `Domain`, `ToolEnv`,
`Database`. Edges: `described_by`, `implemented_by`, `evaluated_on`, `built_by`,
`targets`, `built_on`, `queries`, `cites` (see SPEC §2/§6). `exposes` and `architecture`
are multi-valued **closed-vocab attributes**, NOT edges.

### Pipeline = staged, model-tiered, determinism-bounded

`crawl → parse → classify → resolve(ground) → extract facets → guard/merge/reconcile →
cites → prune → emit` (+ optional `draft profiles`). Module map: `sources`/`parse`
(crawl + entry parsing, no model), `backends` (the model stage), `resolve` (grounding +
OpenAlex/GitHub fetches, no model), `vocab` (closed vocabs + guard), `model` (Graph/Edge),
`pipeline` (assembly), `profiles` (prose), `config`/`log`/`cache` (infra).

Stages are tiered by model need (SPEC §12.1): fetch/parse and cheap edges use **no model**;
**classify** and **facet extraction** use a small model (`Settings.gemini_model`); **prose
drafting** uses the prose tier (`Settings.profile_model`). Expensive edges
(`evaluated_on`/`built_on`/`queries`) carry `{source, evidence}` provenance.

**The backend seam (`backends.py`):** the model-touched stages sit behind one swappable
interface — `classify(entry)`, `extract_facets(entry)`, `draft_profile(prompt)`.
`MockBackend` (default, fully offline, regex classify + table facets) and `GeminiBackend`
(Vertex AI / AI Studio via `google-genai`) return the **same shapes**; vendor is a config
line (`Settings.backend`), not architecture. Each stage's fixed system prompt
(`CLASSIFY_/FACET_/PROFILE_SYSTEM_PROMPT`) is the cache target. `classify` carries a
**biomedical domain gate** — general agent frameworks (CAMEL, AutoGen) → `other`.

**Determinism boundary:** the backend produces *candidates*; `guard_vocab` + the pipeline

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seandavi/biomedical-agent-kg](https://github.com/seandavi/biomedical-agent-kg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
