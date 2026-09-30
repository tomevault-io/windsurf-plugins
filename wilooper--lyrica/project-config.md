---
trigger: always_on
description: Lyrica is a high-performance **Python/Flask REST API** (v1.5.0) that aggregates song lyrics from multiple sources (Genius, LRCLIB, YouTube Music, NetEase, Megalobiz, Musixmatch, Lrcmux, Apple Music) with optional mood analysis, metadata enrichment, trending analytics, word-level and syllable-level sync, multi-tier Redis/memory/disk caching, and real-time lyrics translation/romanization via Groq LLM (`openai/gpt-oss-120b`).
---

# Lyrica — Agent Rules & Project Conventions

## Project Overview

Lyrica is a high-performance **Python/Flask REST API** (v1.5.0) that aggregates song lyrics from multiple sources (Genius, LRCLIB, YouTube Music, NetEase, Megalobiz, Musixmatch, Lrcmux, Apple Music) with optional mood analysis, metadata enrichment, trending analytics, word-level and syllable-level sync, multi-tier Redis/memory/disk caching, and real-time lyrics translation/romanization via Groq LLM (`openai/gpt-oss-120b`).

## Tech Stack

- **Framework**: Flask 3.0+ (with async view support, threaded concurrency, and Vercel/Render/HF deployment compatibility)
- **HTTP Client**: `httpx` (async) with connection pooling (`max_connections=200`) — never `requests`
- **Serialization**: `orjson` (with stdlib `json` fallback)
- **HTML Parsing**: Fast regex and standard HTML decoding — never heavy `bs4` in runtime paths
- **Server**: Gunicorn (`gevent` / `gthread` for production), multi-threaded Flask dev server (local)
- **Python**: 3.11+
- **Config**: `.env` for secrets, `.lyrica.config` (INI format) for user preferences
- **Cache**: Multi-tier caching (`L1 Memory LRU` → `L2 Redis` → `L3 Disk JSON`)

## Architecture

```
lyrica/
├── run.py                  # Entry point (app instance & multi-threaded local runner)
├── pyproject.toml          # Python project & Vercel entrypoint config ([tool.vercel] entrypoint = "run:app")
├── vercel.json             # Vercel serverless routing configuration
├── gunicorn.config.py      # Production Gunicorn configuration (gevent / gthread concurrency)
├── src/
│   ├── __init__.py         # Package version (currently 1.5.0)
│   ├── app.py              # Flask app factory (create_app) + top-level app instance for Vercel
│   ├── router.py           # All route handlers
│   ├── config.py           # Environment variable loading (REDIS_URL, GROQ_MODEL, tokens)
│   ├── user_config.py      # .lyrica.config INI file parser
│   ├── cache.py            # Multi-tier caching system (L1 Memory LRU, L2 Redis, L3 Disk)
│   ├── fetch_controller.py # Orchestrates fetcher sequence & multi-sync-level fallback
│   ├── logger.py           # Centralized logging
│   ├── proxy_manager.py    # Thread-safe round-robin proxy pool singleton
│   ├── groq_key_manager.py # Groq API multi-key round-robin & cooldowns
│   ├── groq_processor.py   # LLM translation/transliteration logic via openai/gpt-oss-120b
│   ├── translation_cache.py# Multi-tier caching for translation responses
│   ├── sources/            # Lyrics source fetchers
│   │   ├── base_fetcher.py # Base class + shared httpx client with connection pooling
│   │   ├── lrclib_fetcher.py
│   │   ├── genius_fetcher.py  # Fast regex/HTML parser without bs4
│   │   ├── youtube_fetcher.py  # 3-layer: ytmusicapi → transcript-api → yt-dlp
│   │   ├── netease_fetcher.py
│   │   ├── megalobiz_fetcher.py
│   │   ├── musixmatch_fetcher.py
│   │   ├── lrcmux_fetcher.py   # Musixmatch via api.lrcmux.dev; line & word-level sync
│   │   └── apple_music_fetcher.py # Apple Music AMP API; line, word-level & syllable-level sync
│   ├── sentiment_analyzer.py
│   ├── metadata_extractor.py # Non-blocking async metadata aggregation (httpx + asyncio.gather)
│   └── trending_analytics.py # Apple Music RSS trending engine (httpx)
├── guide/
│   ├── SETUP_GUIDE.md        # Installation & setup guide
│   ├── USER_GUIDE.md         # Comprehensive API reference guide
│   ├── WORD_SYNC_GUIDE.md    # Word-level sync documentation (schema + implementation examples)
│   ├── TRANSLATION_GUIDE.md  # Detailed guide on translation configuration
│   └── DEPLOYMENT_GUIDE.md   # Deployment guide across Vercel, Docker, VPS, Render, HF, Railway, etc.
├── Test/
│   ├── test_production_upgrades.py      # Automated verification suite for cache, async, trending, routes
│   └── lrcmux/
│       ├── run_test_and_log.py          # Integration test (line + word level)
│       └── test_fetcher_integration.py  # Unit-style fetcher assertions
├── scripts/
│   └── parse_flask_routes.py # OpenAPI 3.0 route parser script
├── .env.example
├── .lyrica.config.example
├── README.md
├── openapi.json
└── requirements.txt
```

## Source Registry

| ID | Name | Fetcher file | Notes |
|----|------|-------------|-------|
| 1 | genius | `genius_fetcher.py` | Requires `GENIUS_TOKEN` (fast regex parser, no bs4) |
| 2 | lrclib | `lrclib_fetcher.py` | Free, very reliable |
| 3 | youtube | `youtube_fetcher.py` | 3-layer fallback |
| 4 | netease | `netease_fetcher.py` | Via syncedlyrics |
| 5 | megalobiz | `megalobiz_fetcher.py` | Via syncedlyrics |
| 6 | musixmatch | `musixmatch_fetcher.py` | Via syncedlyrics, requires `MUSIXMATCH_TOKEN` |
| 7 | lrcmux | `lrcmux_fetcher.py` | Musixmatch via api.lrcmux.dev, no token needed |
| 8 | apple_music | `apple_music_fetcher.py` | Apple Music AMP API, requires `APPLE_MUSIC_DEVELOPER_TOKEN` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wilooper/Lyrica](https://github.com/Wilooper/Lyrica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
