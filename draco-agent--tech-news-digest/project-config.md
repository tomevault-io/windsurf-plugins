---
trigger: always_on
description: Path to GitHub App private key PEM file
---


# Tech News Digest

Automated tech news digest system with unified data source model, quality scoring pipeline, and template-based output generation.

## Quick Start

1. **Configuration Setup**: Default configs are in `config/defaults/`. Copy to workspace for customization:
   ```bash
   mkdir -p workspace/config
   cp config/defaults/sources.json workspace/config/tech-news-digest-sources.json
   cp config/defaults/topics.json workspace/config/tech-news-digest-topics.json
   ```

2. **Environment Variables**: 
   - `TWITTERAPI_IO_KEY` - twitterapi.io API key (optional, preferred)
   - `X_BEARER_TOKEN` - Twitter/X official API bearer token (optional, fallback)
   - `TAVILY_API_KEY` - Tavily Search API key, alternative to Brave (optional)
   - `WEB_SEARCH_BACKEND` - Web search backend: auto|brave|tavily (optional, default: auto)
   - `BRAVE_API_KEYS` - Brave Search API keys, comma-separated for rotation (optional)
   - `BRAVE_API_KEY` - Single Brave key fallback (optional)
   - `GITHUB_TOKEN` - GitHub personal access token (optional, improves rate limits)

3. **Generate Digest**:
   ```bash
   # Unified pipeline (recommended) — runs all 6 sources in parallel + merge
   python3 scripts/run-pipeline.py \
     --defaults config/defaults \
     --config workspace/config \
     --hours 48 --freshness pd \
     --archive-dir workspace/archive/tech-news-digest/ \
     --output /tmp/td-merged.json --verbose --force
   ```

4. **Use Templates**: Apply Discord, email, or PDF templates to merged output

## Configuration Files

### `sources.json` - Unified Data Sources
```json
{
  "sources": [
    {
      "id": "openai-rss",
      "type": "rss",
      "name": "OpenAI Blog",
      "url": "https://openai.com/blog/rss.xml",
      "enabled": true,
      "priority": true,
      "topics": ["llm", "ai-agent"],
      "note": "Official OpenAI updates"
    },
    {
      "id": "sama-twitter",
      "type": "twitter", 
      "name": "Sam Altman",
      "handle": "sama",
      "enabled": true,
      "priority": true,
      "topics": ["llm", "frontier-tech"],
      "note": "OpenAI CEO"
    }
  ]
}
```

### `topics.json` - Enhanced Topic Definitions
```json
{
  "topics": [
    {
      "id": "llm",
      "emoji": "🧠",
      "label": "LLM / Large Models",
      "description": "Large Language Models, foundation models, breakthroughs",
      "search": {
        "queries": ["LLM latest news", "large language model breakthroughs"],
        "must_include": ["LLM", "large language model", "foundation model"],
        "exclude": ["tutorial", "beginner guide"]
      },
      "display": {
        "max_items": 8,
        "style": "detailed"
      }
    }
  ]
}
```

## Scripts Pipeline

### `run-pipeline.py` - Unified Pipeline (Recommended)
```bash
python3 scripts/run-pipeline.py \
  --defaults config/defaults [--config CONFIG_DIR] \
  --hours 48 --freshness pd \
  --archive-dir workspace/archive/tech-news-digest/ \
  --output /tmp/td-merged.json --verbose --force
```
- **Features**: Runs all 6 fetch steps in parallel, then merges + deduplicates + scores
- **Output**: Final merged JSON ready for report generation (~30s total)
- **Metadata**: Saves per-step timing and counts to `*.meta.json`
- **GitHub Auth**: Auto-generates GitHub App token if `$GITHUB_TOKEN` not set
- **Fallback**: If this fails, run individual scripts below

### Individual Scripts (Fallback)

#### `fetch-rss.py` - RSS Feed Fetcher
```bash
python3 scripts/fetch-rss.py [--defaults DIR] [--config DIR] [--hours 48] [--output FILE] [--verbose]
```
- Parallel fetching (10 workers), retry with backoff, feedparser + regex fallback
- Timeout: 30s per feed, ETag/Last-Modified caching

#### `fetch-twitter.py` - Twitter/X KOL Monitor
```bash
python3 scripts/fetch-twitter.py [--defaults DIR] [--config DIR] [--hours 48] [--output FILE] [--backend auto|official|twitterapiio]
```
- Backend auto-detection: uses twitterapi.io if `TWITTERAPI_IO_KEY` set, else official X API v2 if `X_BEARER_TOKEN` set
- Rate limit handling, engagement metrics, retry with backoff

#### `fetch-web.py` - Web Search Engine
```bash
python3 scripts/fetch-web.py [--defaults DIR] [--config DIR] [--freshness pd] [--output FILE]
```
- Auto-detects Brave API rate limit: paid plans → parallel queries, free → sequential
- Without API: generates search interface for agents

#### `fetch-github.py` - GitHub Releases Monitor
```bash
python3 scripts/fetch-github.py [--defaults DIR] [--config DIR] [--hours 168] [--output FILE]
```
- Parallel fetching (10 workers), 30s timeout
- Auth priority: `$GITHUB_TOKEN` → GitHub App auto-generate → `gh` CLI → unauthenticated (60 req/hr)


#### `fetch-github.py --trending` - GitHub Trending Repos
```bash
python3 scripts/fetch-github.py --trending [--hours 48] [--output FILE] [--verbose]
```
- Searches GitHub API for trending repos across 4 topics (LLM, AI Agent, Crypto, Frontier Tech)
- Quality scoring: base 5 + daily_stars_est / 10, max 15

#### `fetch-reddit.py` - Reddit Posts Fetcher
```bash
python3 scripts/fetch-reddit.py [--defaults DIR] [--config DIR] [--hours 48] [--output FILE]
```
- Parallel fetching (4 workers), public JSON API (no auth required)
- 13 subreddits with score filtering


#### `enrich-articles.py` - Article Full-Text Enrichment
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [draco-agent/tech-news-digest](https://github.com/draco-agent/tech-news-digest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
