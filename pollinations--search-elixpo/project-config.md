---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Summary

lixSearch is a multi-service intelligent search assistant (Python/Quart) that searches the web, fetches videos/images, and synthesizes answers with sources. It uses a pipeline architecture with distributed caching, vector embeddings via IPC, and Playwright-based search agents.

## Build & Run

### Docker (production)
```bash
cp .env.example .env          # then fill in TOKEN, MODEL, HF_TOKEN
./deploy.sh build              # build image
./deploy.sh start 3            # start with 3 app containers
./deploy.sh scale 5            # scale to 5
./deploy.sh health             # check all services
./deploy.sh logs app           # tail app logs
./autoscale.sh                 # CPU-based autoscaler daemon (1-5 replicas)
```

### Local development
```bash
source venv/bin/activate       # Python 3.11 venv at /mnt/volume_sfo2_01/lixSearch/venv/
redis-server --port 9530 &     # Redis on custom port
chroma run --host localhost --port 9001 &
APP_MODE=ipc python lixsearch/ipcService/main.py &      # embedding service (port 9510)
APP_MODE=worker WORKER_PORT=9002 python lixsearch/app/main.py &
```

### Tests (integration, no unit test framework)
```bash
python tester/test_multi_turn_session.py
python tester/test_session_persistence.py
python tester/test_redis_semantic_cache.py
```

### Health check
```bash
curl http://localhost:9002/api/health    # direct worker
curl http://localhost/api/health         # via nginx
```

## Architecture

### Network topology

All internal services communicate over a private Docker network (`elixpo-network`). None of the internal 9xxx ports are published to the host — they are only reachable within the Docker network (via `expose`). The only externally reachable entry point is nginx.

```
Internet → Cloudflare → search.elixpo.com
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
     Next.js frontend (/)            nginx (:10001, API key)
     search.elixpo/                  ┌────────────────────────┐
     ├── /c/[session]               │  :80   internal/dev    │
     ├── /discover                  │  :10001 production     │
     ├── /library                   │  :443   reserved       │
     └── /api/* (proxy)             └──────────┬─────────────┘
              │                                │
              │  BACKEND_URL + X-Internal-Key   │ proxy_pass
              └───────────┬────────────────────┘
                          ▼
               lixsearch-app (:9002, N replicas, 10 Hypercorn workers each)
                 ├── Gateways (app/gateways/*.py)
                 ├── Pipeline (pipeline/lixsearch.py) — main async generator
                 │     ├── Tool execution (pipeline/optimized_tool_execution.py)
                 │     ├── RAG (ragService/)
                 │     └── LLM inference → Pollinations API (external)
                 ├── Session/cache (sessions/, ragService/cacheCoordinator.py)
                 └── IPC client → ipc-service (:9510, singleton)
                                    ├── CoreEmbeddingService (sentence-transformers)
                                    ├── SearchAgentPool (Playwright text + image agents)
                                    └── Chroma vector DB (:9001)
Redis (:9530) shared across all containers:
  DB 0 — semantic query cache (5min TTL, per-session)
  DB 1 — URL embedding cache (24h TTL, global)
  DB 2 — session hot window (30min TTL, 20 msgs, LRU evicted to disk)
```

### Ports & exposure

| Service | Internal port | Published to host? | Purpose |
|---------|--------------|-------------------|---------|
| nginx | 80, 443, 10001 | **Yes** — 80:80, 443:443, 10001:10001 | Only external entry point |
| lixsearch-app | 9002 | No (`expose` only) | Quart API workers |
| ipc-service | 9510 | No (`expose` only) | Embedding model + Playwright pool |
| chroma-server | 9001 | No (`expose` only) | Vector DB |
| redis | 9530 | No (`expose` only) | Cache (3 DBs) |

### Authentication

- **Port 80** (nginx internal server): No API key required. Used for local dev and inter-service health checks.
- **Port 10001** (nginx authenticated server): API key required via `X-API-Key` header or `?key=` query param. This is the production endpoint routed from `search.elixpo.com` through Cloudflare.
- **Auth-exempt paths** (both ports): `/api/health`, `/docs`, `/api/docs`, `/openapi.json`, `/openapi.yaml`
- **App-level auth**: `INTERNAL_API_KEY` env var in `before_request` middleware (`app/main.py`). Separate from the nginx API key — used for internal service-to-service calls.

### Request flow
```
User → Cloudflare → nginx:10001 (API key check) → app:9002 → gateway (search.py / chat.py)
  → run_elixposearch_pipeline() async generator → query decomposition → tool routing
  → web_search/fetch/image_search/youtube → RAG retrieval → semantic cache check
  → LLM synthesis → SSE or JSON response
```

### Conversation storage (two-tier hybrid)
- **Hot**: Redis DB 2 — last 20 messages per session
- **Cold**: Huffman-compressed `.huff` files in `./data/conversations/<session_id>.huff`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pollinations/search.elixpo](https://github.com/pollinations/search.elixpo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
