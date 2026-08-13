---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TrendiaTR is a live Turkish news aggregation and trend-scoring platform running at **trendiatr.com**. It ingests from 50+ RSS feeds, 37 Telegram channels, and X (Twitter) trends, clusters semantically related articles using vector embeddings, and scores clusters with a proprietary TPS metric. The system is fully Dockerized and **always running in production** — treat every DB/infrastructure change with care.

## Common Commands

```bash
# Start infrastructure + web (no workers)
sudo docker compose up -d

# Start everything including workers
sudo docker compose --profile workers up -d

# Rebuild images. ALWAYS pass --profile workers: without it, compose builds only
# api_server, dashboard and db_init and silently skips the other 10 services —
# no error, no warning. They stay on the old image while the built three move on.
sudo docker compose --profile workers build
sudo docker compose --profile workers up -d

# Restart the API server (also runs DB migrations via init_db())
sudo docker compose restart api_server

# Restart a specific worker
sudo docker compose restart ttw_gravity   # gravity_worker
sudo docker compose restart ttw_summarizer

# View logs
sudo docker compose logs api_server --tail=50 -f
sudo docker compose logs ttw_gravity --tail=30

# Tests that import app modules run INSIDE a container. The host has pytest but
# none of the app dependencies, so a host run dies on `import sqlalchemy`.
sudo docker exec ttw_api python3 -m pytest tests/test_b2b_api.py -v
sudo docker exec ttw_gravity python3 -m pytest tests/test_scoring_queue_atomicity.py -v

# tests/test_gravity_pagination.py imports nothing from app/ and runs on the host:
python3 -m pytest tests/test_gravity_pagination.py -v

# Run a single test class
sudo docker exec ttw_api python3 -m pytest tests/test_b2b_api.py::TestAuthentication -v

# pytest ships via requirements.txt, so it is only present after an image build:
#   sudo docker compose build api_server && sudo docker compose up -d api_server
# tests/test_summary_analysis_filter.py also runs without pytest:
sudo docker exec ttw_summarizer python3 tests/test_summary_analysis_filter.py

# Syntax-check Python files without running them
python3 -c "import ast; ast.parse(open('app/api/routes.py').read()); print('OK')"
```

## Images & Rebuilds

All 13 services build from the **same `Dockerfile` and the same context** — the
images differ only in tag. So the first service built pays the full cost and the
rest are cache hits (~40s for the other 10).

Application code is bind-mounted, so a code change needs only `git pull` +
`restart`. A rebuild is required only when `requirements.txt` changes.

**`requirements.txt` changes are expensive.** The Dockerfile does
`COPY requirements.txt` before `RUN pip install`, so any edit invalidates that
layer and torch + sentence-transformers + chromadb reinstall from scratch — a
~26 GB layer and several minutes. Check `df -h /` first: the server sits around
83-88 % full, and a build that runs the disk out takes production down with it.
`sudo docker builder prune -f` frees cache but makes the next rebuild pay the
full torch download again.

## Database Migrations

**Schema changes go in `app/database/models.py` — two places:**
1. Add the `Column(...)` to the ORM class
2. Add a migration block in `init_db()` using the existing pattern:
   ```python
   if 'new_column' not in trend_columns:
       conn.execute(text("ALTER TABLE trends ADD COLUMN new_column TYPE"))
   ```

`init_db()` is called automatically at `api_server` startup. Only adds columns, never drops data. After adding a migration, `sudo docker compose restart api_server` applies it.

## Architecture

### Request → Response Path (Web)

```
Browser → Nginx → ttw_api (Flask/Gunicorn 4 workers)
              → web_server.py::create_app()
              → api_bp (routes.py)    — HTML pages + JSON utility endpoints
              → api_v1_bp (api_v1.py) — B2B REST API (/api/v1/)
              → api_admin_b2b_bp      — API client management
```

### Ingestion → Scoring Pipeline

```
rss_fetcher.py / telegram_bot.py / social_worker.py
    → text_utils.clean_text()
    → classifier.fast_classify()
    → ai_engine.process_news()         ← embeds text, queries ChromaDB
          → auto-merge OR ask Ollama (Qwen 2.5:1.5b)
    → DB write (Trend + RawNews + TrendArrivals)
    → scoring_queue.enqueue(trend_id)  ← Redis priority queue

gravity_worker.py (every 5s)
    → TPSCalculator.run_tps_cycle()
    → threshold checks → alert_service or publish_to_channel()
```

### Key Modules

| File | Role |
|------|------|
| `app/core/ai_engine.py` | Central AI pipeline: embed → ChromaDB search → Ollama verify |
| `app/core/scoring.py` | `TPSCalculator` — V/E/S/N signals + criticality boost |
| `app/core/translation.py` | Turkish→Persian translation: Redis → DB → Gemini |
| `app/api/routes.py` | All Flask routes (~2300 lines); contains `invalidate_trend_caches()` |
| `app/database/models.py` | All SQLAlchemy models + `init_db()` migration runner |
| `app/workers/summarizer.py` | Gemini summarization; logs to `ai_monitor_data.csv` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pdrm55/turkey-trend-watcher](https://github.com/pdrm55/turkey-trend-watcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
