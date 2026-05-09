---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

AI News Aggregator - A Python-based multi-agent pipeline that collects AI/ML news from multiple sources (RSS feeds, arXiv API, Twitter, Reddit, Bluesky, Mastodon), analyzes them using Codex Opus 4.7 with adaptive thinking, and serves a modern Svelte SPA frontend with AATF branding.

**Testing:** The user always runs tests themselves. Do not run the pipeline or tests unless explicitly asked.

## Commands

### Docker (Production)
```bash
docker-compose build                    # Build container
docker-compose up -d                    # Start services (serves existing content only)
docker-compose down                     # Stop services
docker logs ai-news-aggregator          # View container logs

# Manual pipeline run (trigger data collection)
docker exec ai-news-aggregator python3 /app/run_pipeline.py --config-dir /app/config --data-dir /app/data --web-dir /app/web

# Enable scheduled collection (cron)
ENABLE_CRON=true docker-compose up -d
```

### Local Development (Pipeline)
```bash
source venv/bin/activate                            # Activate virtual environment
pip install -r requirements.txt                     # Install dependencies
python3 run_pipeline.py --create-config             # Generate default config
python3 run_pipeline.py --config-dir ./config --data-dir ./data --web-dir ./web

# Run for a specific date (useful for testing/backfilling)
TARGET_DATE="2026-01-02" python3 run_pipeline.py --config-dir ./config --data-dir ./data --web-dir ./web

# Resume after a crash (auto-detect latest checkpoint)
python3 run_pipeline.py --resume --config-dir ./config --data-dir ./data --web-dir ./web

# Resume from a specific phase (loads earlier phases from checkpoint)
python3 run_pipeline.py --resume-from 3 --config-dir ./config --data-dir ./data --web-dir ./web
```

### Frontend Development
```bash
cd frontend
npm install                     # Install dependencies
npm run dev                     # Start dev server at http://localhost:5173
npm run build                   # Build production (outputs to ../web)
npm run preview                 # Preview production build
npm run check                   # TypeScript type checking
```

There are no unit tests, linting, or type checking configured.

## Architecture

### Multi-Agent Pipeline (run_pipeline.py)

```
Phase 0: Ecosystem Context Initialization
    ↓
Phase 1: Parallel Gathering (4 gatherers)
    ↓
Phase 2: Parallel Analysis (4 analyzers with grounding context)
    ↓
Phase 3: Cross-Category Topic Detection (ULTRATHINK)
    ↓
Phase 4: Executive Summary Generation
    ↓
Phase 4.5: Link Enrichment (adds internal links to summaries)
    ↓
Phase 4.6: Ecosystem Enrichment (detect new model releases)
    ↓
Phase 4.7: Hero Image Generation (Gemini 3 Pro via configured provider)
    ↓
Phase 5: Assembly & Output
    ↓
Phase 6: JSON Data Generation (for SPA frontend)
    ↓
Phase 6.5: RSS Feed Generation (Atom 1.0 with Media RSS)
    ↓
Phase 7: Search Index Update (Lunr.js compatible)
```

### Agent Pairs

| Agent Pair | Gatherer Sources | Analysis Focus |
|------------|------------------|----------------|
| **News** | RSS feeds + articles from Twitter links | Product releases, company news |
| **Research** | arXiv API + research blogs (LessWrong) | Research findings, breakthroughs |
| **Social** | Twitter, Bluesky, Mastodon | Industry discussions, reactions |
| **Reddit** | Reddit JSON API | Community discussions, debates |

### Directory Structure

```
agents/
├── __init__.py
├── llm_client.py              # Anthropic client with extended thinking
├── base.py                    # Base classes (BaseGatherer, BaseAnalyzer)
├── orchestrator.py            # Main coordinator
├── link_enricher.py           # Adds internal links to summaries
├── cost_tracker.py            # LLM API cost tracking
├── phase_tracker.py           # Phase status tracking and end-of-run summary
├── ecosystem_context.py       # AI model release tracking for grounding
├── gatherers/
│   ├── news_gatherer.py       # RSS + Twitter-linked articles
│   ├── research_gatherer.py   # arXiv + research blogs (LessWrong)
│   ├── social_gatherer.py     # Twitter, Bluesky, Mastodon (with status tracking)
│   ├── reddit_gatherer.py     # Reddit
│   └── link_follower.py       # Smart link extraction from social posts
└── analyzers/
    ├── news_analyzer.py
    ├── research_analyzer.py
    ├── social_analyzer.py
    └── reddit_analyzer.py

generators/
├── json_generator.py          # Generates JSON data for SPA frontend
├── search_indexer.py          # Builds Lunr.js search index
├── hero_generator.py          # Daily hero image with skunk mascot
└── feed_generator.py          # Atom RSS feeds with Media RSS support

scripts/
└── regenerate_hero.py         # Manual hero image regeneration

assets/
└── skunk-reference.png        # AATF skunk mascot reference image

frontend/                       # Svelte SPA frontend
├── src/
│   ├── lib/
│   │   ├── components/        # Svelte components
│   │   ├── stores/            # State management
│   │   ├── services/          # Data loading, search
│   │   └── types/             # TypeScript types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flyryan/ai-news-aggregator](https://github.com/flyryan/ai-news-aggregator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
