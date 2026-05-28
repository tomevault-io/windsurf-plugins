---
trigger: always_on
description: Self-hosted AI infrastructure stack. Each service is an isolated
---

# AI Agent Instructions — yai

Self-hosted AI infrastructure stack. Each service is an isolated
docker-compose project under its own folder, managed by a single CLI
(`./yai.sh`). All data is bind-mounted to `./<service>/data/` and survives
container restarts.

## Architecture

```
       ┌──────────────────────────┐   ┌──────────────────────────┐
       │  LLM + observability     │   │  Browser fleet           │
       │  litellm (gateway)       │   │  browserless (generic)   │
       │  langfuse (traces/evals) │   │  firecrawl (crawl API)   │
       └──────────────────────────┘   └──────────────────────────┘
                          │                       │
                          ▼                       ▼
       ┌──────────────────────────────────────────────────────────┐
       │  Workflow & data                                         │
       │  n8n  •  windmill  •  qdrant  •  minio  •  postgres      │
       └──────────────────────────────────────────────────────────┘
                                  │
                                  ▼
       ┌──────────────────────────────────────────────────────────┐
       │  Observability  (all on yai-infra Docker network)       │
       │  grafana  •  vmetrics  •  vlogs  •  vtraces  •  vector │
       └──────────────────────────────────────────────────────────┘
```

Every service is reachable on its direct host port. Traefik also exposes all
services via `<service>.localhost:27000` hostname routing and a navigation
dashboard at `http://localhost:27000`.

## Services

| Service | Image (pinned) | Host ports | Description |
|---|---|---|---|
| `postgres` | `postgres:17.10` | 25432 | Shared Postgres 17 — used by LiteLLM and ad-hoc workloads. Other stacks bring their own embedded DB. |
| `minio` | `pgsty/minio:RELEASE.2026-04-17T00-00-00Z` | 25000 (S3), 25001 (console) | S3-compatible object storage. (Upstream `minio/minio` was archived 2026-04-25; pgsty/minio is the maintained fork used here.) |
| `qdrant` | `qdrant/qdrant:v1.13.1` | 26000 (REST), 26001 (gRPC) | Vector DB for RAG / embeddings. |
| `browserless` | `ghcr.io/browserless/chromium:latest` | 26003 | Generic headless Chromium API (Puppeteer / Playwright / CDP). Use for scripted scraping. |
| `firecrawl` | `ghcr.io/firecrawl/firecrawl:latest` | 21000 | Web scraping & crawling API (built-in queues, Playwright backend). No versioned GHCR tags published; `:latest` is the only option. |
| `n8n` | `docker.n8n.io/n8nio/n8n:2.21.3` | 26002 | Workflow automation (queue mode, postgres + redis + workers + runners). |
| `litellm` | `litellm/litellm:v1.85.0` | 24000 | OpenAI-compatible LLM gateway. Uses shared `postgres`. |
| `langfuse` | `langfuse/langfuse:3` (+worker, clickhouse, minio, redis, postgres) | 23000 (web), 23090 (media MinIO) | LLM observability — traces, evals, prompt mgmt. |
| `windmill` | `ghcr.io/windmill-labs/windmill:1.703.0` | 28000 | Workflow & script engine (postgres + 3 workers + native worker). |
| `vmetrics` | `victoriametrics/victoria-metrics:v1.143.0` | 28428 | Metrics store + built-in scraper (PromQL/MetricsQL). Scrape targets at `/targets`. |
| `vlogs` | `victoriametrics/victoria-logs:v1.50.0` | 29428 | Log store (Loki-compatible push, LogsQL query). |
| `vtraces` | `victoriametrics/victoria-traces:v0.8.2` | 21428 | Trace store (OTLP ingest, Jaeger-compatible query). |
| `vector` | `timberio/vector:0.44.0-debian` | _(none, internal)_ | Docker log collector — ships `yai-*` container logs to vlogs via Loki push. Exposes Prometheus metrics on `:9598` (scraped by vmetrics). |
| `node-exporter` | `prom/node-exporter:v1.9.1` | _(none, internal)_ | Prometheus Node Exporter — host OS metrics (CPU, memory, disk, network). Scraped by vmetrics as job `node_exporters`. |
| `grafana` | `grafana/grafana:12.4.3` | 22000 | Grafana UI. Queries obs services via the `yai-infra` Docker network. |
| `traefik` | `traefik:v3.3` (+ `nginx:1.27-alpine`) | 80 (HTTP), 27001 (API) | HTTP reverse proxy. Routes `*.localhost` and `*.yai.orb.local`; Prometheus metrics at `:27001/metrics`. |

## Directory layout

```
yai/
├── yai.sh                     # Management CLI
├── AGENTS.md  CLAUDE.md       # Agent rules
├── README.md
├── .gitignore
└── <service>/
    ├── docker-compose.yml
    ├── .env                   # Placeholder secrets (`change_me`) — replace before start
    ├── AGENTS.md              # Per-service docs + upstream links
    └── data/                  # Bind-mounted runtime state (gitignored)
```

Each service folder also supports an optional `.env.local` file that
overrides values from `.env` — gitignored, ideal for personal/dev tweaks.

## Management — `yai.sh`

```
yai stack <cmd>              operate on every service
yai service <name> <cmd>     operate on one service
yai <cmd> [service|all]      short form (same semantics as ydocker/server.sh)

Commands: init | start | stop | restart | logs | ps | status

Examples:
  ./yai.sh stack start
  ./yai.sh service n8n logs
  ./yai.sh service langfuse restart
  ./yai.sh start qdrant
  ./yai.sh stop all
```

### First-time setup

1. `./yai.sh init all` — creates every `./data/` tree and warns on
   `change_me` placeholders.
2. Edit each `<service>/.env` and replace every `change_me` value.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ynfra/yai](https://github.com/ynfra/yai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
