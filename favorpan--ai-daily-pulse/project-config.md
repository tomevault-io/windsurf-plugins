---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI 信息雷达 (AI Info Aggregator) — a Python pipeline that fetches articles from 43+ RSS feeds, scores them with an LLM, deduplicates, generates Chinese summaries, and outputs Obsidian-compatible daily digest Markdown files.

**Key audience focus:** OPC/solo-founder AI money-making cases, AI+ecommerce, AI tool workflows, AI new tech/models, and funding rounds.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the full pipeline (requires API key via env var or config.toml)
export API_KEY=sk-...
python main.py

# Run with custom lookback window (default: 1 day)
LOOKBACK_DAYS=3 python main.py

# Run tests (80 tests across 8 files)
pytest

# Run a single test
pytest tests/test_history.py::test_filter_unseen_skips_recent_match

# Web frontend local dev (requires digest JSON in output/)
cd web && npm install && npm run dev
# Opens at http://localhost:3000
```

## Architecture

Full pipeline: **fetch → prefilter (rules) → history dedup (URL) → score (LLM, concurrent) → dedup (Jaccard + LLM) → summarize_zh (concurrent) → summarize_en (concurrent) → why_now (score ≥ 7) → trend detection (tags clustering) → insights (build directions + social posts) → write (Markdown + JSON)**

1. [main.py](main.py) — Entry point. Loads config, orchestrates the pipeline, prints run summary with timing breakdown.
2. [src/config.py](src/config.py) — Loads `config.toml` with defaults, overrides with env vars. Single `load_config()` returns a flat dict.
3. [src/feeds.py](src/feeds.py) — Loads `feeds.toml`, fetches RSS via feedparser with concurrent workers (ThreadPoolExecutor), filters by lookback window, extracts and cleans HTML content. Also performs rule-based prefiltering: drops titles < 5 chars, content < 100 chars without AI keywords.
4. [src/feed_health.py](src/feed_health.py) — Tracks per-feed health in `data/feed_health.json`: last success/failure time, consecutive failures, cumulative article count. WARNING logged when ≥ 3 consecutive failures.
5. [src/history.py](src/history.py) — Manages `data/pushed.json` for 90-day URL dedup window. Pure functions, no side effects except file I/O in `save_history`.
6. [src/scorer.py](src/scorer.py) — Core LLM interactions: scoring (JSON mode with topic-specific rubrics), Jaccard title dedup pre-filter (threshold 0.4), LLM precise dedup, Chinese summaries, English summaries, and "why now" commentary (for score ≥ 7). All with retry logic (3 attempts, exponential backoff). Uses OpenAI-compatible SDK with configurable `base_url`.
7. [src/trends.py](src/trends.py) — Trend detection via LLM tag cross-source clustering. Builds keyword→source mapping from article tags, flags keywords appearing in ≥ 3 distinct sources as "strong signals." Assigns confidence levels: high (5+), medium (3+), low.
8. [src/insights.py](src/insights.py) — Two sub-features:
   - **Build directions:** Calls last30days CLI to search Reddit/HN for community discussion data matching today's articles, picks top 2 by engagement, generates project suggestions via LLM (with difficulty, MVP days, monetization).
   - **Social posts:** Generates bilingual X/Twitter posts (≤ 280 chars) and X Thread (3-5 tweets) via LLM.
9. [src/writer.py](src/writer.py) — Generates Obsidian Markdown (`AI Daily - YYYY-MM-DD.md`) grouped by topic in `TOPIC_ORDER`, plus structured JSON digest (`digest-YYYY-MM-DD.json` + `latest.json`) for the web frontend. Article IDs are SHA-256 URL hashes (12-char hex).
10. [feeds.toml](feeds.toml) — All feed sources (~47). Each entry has `name`, `url`, `lang` (en/zh).
11. [config.toml](config.toml) — AI model, pipeline, and last30days configuration. See "Configuration" below.
12. [data/pushed.json](data/pushed.json) — History file tracking pushed URLs with timestamps. Auto-pruned to 90-day window.
13. [data/feed_health.json](data/feed_health.json) — Per-feed health metrics updated on each run.

## Configuration

All AI and pipeline settings live in `config.toml`. Env vars override config file values.

Key fields:
- `api.api_key` — API key (env: `API_KEY`)
- `api.base_url` — API endpoint (default: `https://api.deepseek.com`)
- `api.scoring_model` / `api.summary_model` — Model IDs (default: `deepseek-v4-flash`)
- `api.price_in_per_m` / `api.price_out_per_m` — Token pricing for cost calculation
- `pipeline.fetch_workers` — RSS concurrency (default: 8)
- `pipeline.score_workers` — Scoring/summary concurrency (default: 4)
- `pipeline.dedup_window_days` — History dedup window (default: 90)
- `pipeline.content_cap` — Max chars per article sent to LLM (default: 4000)
- `pipeline.fetch_timeout` — RSS request timeout in seconds (default: 15)
- `last30days_enabled` — Enable community discussion search (default: false, requires last30days CLI)
- `last30days_engine_path` / `last30days_python_path` / `last30days_timeout` / `last30days_search_sources` — last30days subprocess config

Works with any OpenAI-compatible API: DeepSeek, OpenAI, Ollama, etc.

## Scoring System

- Articles scored 0-10 by configured LLM using topic-specific rubrics defined in [src/scorer.py](src/scorer.py)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FavorPan/ai-daily-pulse](https://github.com/FavorPan/ai-daily-pulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
