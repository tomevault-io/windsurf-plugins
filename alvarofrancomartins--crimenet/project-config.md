---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

CRIMENET is a knowledge graph of global organized crime (4,505 orgs, 10,935 relationships) extracted from 1,418 Wikipedia articles via DeepSeek LLM. Every edge carries a verbatim evidence quote, a description, a time period, and a versioned source URL. The web app is fully static, served from `/crimenet/` at alvarofrancomartins.com/crimenet.

The pipeline fetches 1,491 articles (`data/articles.csv` + `data/txts/`), but some — concept articles, non-org subjects — yield no nodes. The 1,418 figure is the number of articles that actually contributed data to the graph, computed as unique article URLs across all nodes' `own_sources` and `mentioned_in`. This is the real source footprint and the number used everywhere in the app and docs.

## Key commands

```bash
# One-time
pip install -r requirements.txt
export DEEPSEEK_API_KEY="sk-..."

# Full pipeline (steps 0→4, stops on first failure, ends at data/crimenet_raw.json — does NOT build app)
cd pipeline && python run_pipeline.py && cd ..

# Individual pipeline steps (from pipeline/ dir, all re-runnable independently)
python 1_fetch_wikipedia.py --csv ../data/articles.csv --output ../data/txts           # re-fetch all
python 1_fetch_wikipedia.py -F "Jalisco" --csv ../data/articles.csv --output ../data/txts --force  # single article
python 2_extract_network.py --dir ../data/txts --force-failed                           # retry partials only
python 4_merge.py --dir ../data/txts --output ../data/crimenet_raw.json         # rebuild dataset (no API calls)

# Build app assets from data/crimenet.json (ORDER MATTERS)
python build/build_compact_data.py        --input data/crimenet.json --output app/data/compact.json
python build/build_adjacency.py        --input data/crimenet.json --output app/data/crimenet_adj.json
python build/build_evidence_shards.py  --input data/crimenet.json --output app/data/evidence
python build/build_relationship_summaries.py --input data/crimenet.json --output app/data/relationship_summaries
python build/build_communities_data.py --input data/crimenet.json --output app/data/communities.json --workers 10
python build/build_bridges_data.py --input data/crimenet.json --communities app/data/communities.json --output app/data/bridges.json
python build/build_triadic_data.py --input data/crimenet.json --output app/data/triadic_signals.json
python build/build_centrality.py         --input data/crimenet.json --output app/data/centrality.json
python build/build_static_pages.py     --input data/crimenet.json --output app --base-url https://www.alvarofrancomartins.com/crimenet

# Local preview (must use HTTP — fetch() fails on file://)
python -m http.server 8000   # then open localhost:8000/app/index.html

# Audit (1. audit 0–5 → 2. review → 3. apply)
python audit/0_review_wrong_merges.py && python audit/1_review_missed_merges.py && python audit/2_review_edges.py && python audit/3_review_country_links.py && python audit/4_review_umbrella_orgs.py && python audit/5_review_non_criminal_orgs.py
python audit/6_review_suggestions.py   # LLM second opinion → audit_data/6_llm_verdicts.json (run before apply)
python audit/7_apply_corrections.py      # auto-applies confident audit suggestions + manual overrides → data/crimenet.json
#    If you spot an error, add a fix to audit/curated_corrections.py — it always wins over auto.
```

## Architecture

### Pipeline (pipeline/) — Wikipedia → crimenet_raw.json

Each step writes to a distinct output; any step can be re-run independently without invalidating others. Steps 1–4 are resumable (skip existing unless `--force`). Step 2 runs 50 parallel workers against DeepSeek.

- **Step 0**: plain URLs → versioned URLs with `oldid` (articles.csv)
- **Step 1**: fetch Wikipedia HTML via MediaWiki API, extract clean body text + infobox → `data/txts/<folder>/{content.txt,url.txt}`
- **Step 2**: LLM extracts org nodes + edges per article → `article_graph.json` (chunked by ~2500 words, infobox appended to every chunk)
- **Step 3**: LLM enriches profiled orgs (description, aliases, country, country_links, time_period, is_defunct) → `org_profile.json`. Two passes: Pass 1 profiles the subject org; Pass 2 extracts country footprints
- **Step 4**: merges all graph fragments, runs auto-dedup (fuzzy + containment), attaches org profiles, normalizes countries via `lib/countries.py` → `data/crimenet_raw.json`

Step 4 does NOT apply any hand-curated corrections. Run `audit/7_apply_corrections.py` afterward to produce the final `data/crimenet.json`.

**Critical:** Confident audit suggestions are auto-applied by `7_apply_corrections.py`. For manual fixes, add entries to `audit/curated_corrections.py` — they always win over auto-suggestions on conflict. The `6_review_suggestions.py` step is a second opinion that writes `audit_data/6_llm_verdicts.json` and never edits `curated_corrections.py`; `7_apply_corrections.py` reads that report and lets its high-confidence rejections veto the matching auto wrong-merge (BLOCKLIST) and duplicate (KNOWN_DUPLICATES) suggestions only.

### Build (build/) — crimenet.json → static app/


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alvarofrancomartins/CRIMENET](https://github.com/alvarofrancomartins/CRIMENET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
