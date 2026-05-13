---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZotWatch is a personalized academic paper recommendation system that builds a research interest profile from a user's Zotero library and continuously monitors academic sources for relevant new publications. It supports AI-powered summaries, incremental embedding computation, and runs daily via GitHub Actions to output RSS/HTML feeds.

## Commands

```bash
# Install dependencies
uv sync

# Build/rebuild user profile from Zotero library (full rebuild)
uv run zotwatch profile --full

# Incremental profile update (only new/changed items)
uv run zotwatch profile

# Daily watch: fetch candidates, score, and generate RSS + HTML + AI summaries
uv run zotwatch watch

# Only generate RSS feed
uv run zotwatch watch --rss

# Only generate HTML report
uv run zotwatch watch --report

# Custom recommendation count
uv run zotwatch watch --top 50

# Push top recommendations back to Zotero
uv run zotwatch watch --push
```

## Architecture

### Pipeline Flow

1. **Ingest** (`pipeline/ingest.py`): Fetches items from Zotero Web API, stores in SQLite
2. **Profile Build** (`pipeline/profile.py`): Vectorizes library items using Voyage AI API (voyage-3.5), builds FAISS index, extracts top authors/venues
3. **Candidate Fetch** (`pipeline/fetch.py`): Pulls recent papers from Crossref and arXiv
4. **Deduplication** (`pipeline/dedupe.py`): Filters out papers already in the user's library
5. **Scoring** (`pipeline/score.py`): Ranks candidates using weighted combination of similarity, recency, citations, journal quality, and whitelist bonuses
6. **Summarization** (`llm/summarizer.py`): Generates AI summaries via OpenRouter API
7. **Output** (`output/rss.py`, `output/html.py`): Generates RSS feed and/or HTML report

### Directory Structure

```
src/zotwatch/
├── core/               # Core models, protocols, exceptions
├── config/             # Configuration loading and settings
├── infrastructure/     # External service integrations
│   ├── storage/        # SQLite storage
│   ├── embedding/      # Voyage AI + FAISS
│   └── http/           # HTTP client
├── sources/            # Data sources (arXiv, Crossref, Zotero)
├── llm/                # LLM integration (OpenRouter, summarizer)
├── pipeline/           # Processing pipeline (ingest, profile, fetch, dedupe, score)
├── output/             # Output generation (RSS, HTML, push to Zotero)
├── cli/                # Click CLI
└── utils/              # Utilities (logging, datetime, hashing, text)
```

### Key Data Artifacts

- `data/profile.sqlite`: SQLite database storing Zotero items and embeddings
- `data/faiss.index`: FAISS vector index for similarity search
- `data/profile.json`: Profile summary with top authors, venues, and centroid vector
- `data/embeddings.sqlite`: Embedding cache for reusing computed vectors

### Configuration Files (config/)

- `config.yaml`: Unified configuration file containing all settings:
  - `zotero`: Zotero API settings (user_id uses `${ZOTERO_USER_ID}` env var expansion)
  - `sources`: Data source toggles and parameters (days_back, categories, max_results)
  - `scoring`: Score weights, thresholds, decay settings, author/venue whitelists
  - `embedding`: Embedding model configuration (provider, model, batch_size)
  - `llm`: LLM configuration for AI summaries (provider, model, retry settings)
  - `output`: RSS and HTML output settings
  - `watch`: Watch pipeline settings (recent_days, preprint ratio, top_k)

For detailed configuration guides (embedding provider switching, LLM provider switching, threshold modes, etc.), see the "配置指南" section in README.md.

### Configuration Options

#### Dynamic Thresholds (`scoring.thresholds`)

Controls how papers are labeled as `must_read`, `consider`, or `ignore`:

- `mode`: Threshold computation mode
  - `"fixed"`: Use static threshold values (default behavior)
  - `"dynamic"`: Compute thresholds from score distribution per batch
- `must_read`: Fixed threshold for must_read label (default: 0.75)
- `consider`: Fixed threshold for consider label (default: 0.55)
- `dynamic`: Settings for dynamic mode
  - `must_read_percentile`: Top N% marked as must_read (default: 95, meaning top 5%)
  - `consider_percentile`: Percentile for consider threshold (default: 70)
  - `min_must_read`: Minimum score for must_read even in dynamic mode (default: 0.60)
  - `min_consider`: Minimum score for consider even in dynamic mode (default: 0.40)

#### Watch Pipeline (`watch`)

Controls the watch command behavior:

- `recent_days`: Filter papers older than N days (default: 7)
- `max_preprint_ratio`: Maximum ratio of preprints in final results (default: 0.9)
- `top_k`: Default number of recommendations (default: 20)
- `require_abstract`: Filter out candidates without abstracts (default: true)

### Switching Embedding Providers

ZotWatch supports two embedding providers:
- **Voyage AI**: High-quality embeddings, recommended for English papers
- **DashScope** (Alibaba Cloud): Alternative provider with Chinese language support

**Important:** When `scoring.interests.enabled=true`, both `embedding.provider` and `scoring.rerank.provider` MUST use the same provider.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kenanking/ZotWatch](https://github.com/kenanking/ZotWatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
