---
trigger: always_on
description: This file is the single source of truth for an autonomous agent installing and running this repository locally. Follow steps in order. Each step has a verification command — do not proceed until it passes.
---

# AGENTS.md — End-to-End Install Guide for Agents

This file is the single source of truth for an autonomous agent installing and running this repository locally. Follow steps in order. Each step has a verification command — do not proceed until it passes.

The default deployment uses Docker Compose with the `local` profile: PostgreSQL + Trino + Schema Service + Nexus API + Frontend, all on the host network at well-known ports.

---

## 1. Prerequisites

Required on the host:

| Tool | Minimum version | Verify |
|---|---|---|
| Docker Engine | 20.10 | `docker --version` |
| Docker Compose | v2 (`docker compose`) or v1 (`docker-compose`) | `docker compose version` |
| Bash | 4+ | `bash --version` |
| Free RAM for Docker | 8 GB | Docker Desktop → Settings → Resources |
| Free disk | 10 GB | `df -h` |

These host ports must be free: **3000, 5432, 8000, 8001, 8081**. Verify with:

```bash
for p in 3000 5432 8000 8001 8081; do lsof -iTCP:$p -sTCP:LISTEN -n -P || echo "port $p free"; done
```

If any port is in use, stop the offending process before continuing.

---

## 2. Clone (skip if already in the repo)

```bash
git clone https://github.com/actyze/dashboard.git
cd dashboard
```

Verify:

```bash
test -f docker/docker-compose.yml && test -f docker/env.example && echo OK
```

---

## 3. Configure environment

```bash
cd docker
cp env.example .env
```

Note: the template is `env.example` (no leading dot), not `.env.example`. Some older docs say otherwise — trust this file.

Edit `.env` and set, at minimum:

| Variable | What to set | Notes |
|---|---|---|
| `ANTHROPIC_API_KEY` | a real key | Default LLM. Or use `OPENAI_API_KEY` / `GEMINI_API_KEY` / `GROQ_API_KEY` / `PERPLEXITY_API_KEY` and update `EXTERNAL_LLM_MODEL` accordingly |
| `EXTERNAL_LLM_MODEL` | model id matching the key (default `claude-sonnet-4-20250514` works with Anthropic) | |
| `POSTGRES_PASSWORD` | any non-empty string | Replace `your-secure-password-here` |
| `SCHEMA_SERVICE_KEY` | any 32+ char random string | Shared secret between Nexus and Schema Service |

If the agent has no API key available, ask the user — do not fabricate one. The stack will start without a valid key but SQL generation will fail at runtime.

Verify the file is populated:

```bash
grep -E '^(ANTHROPIC_API_KEY|OPENAI_API_KEY|GEMINI_API_KEY|GROQ_API_KEY|PERPLEXITY_API_KEY)=.+' .env \
  && grep -E '^POSTGRES_PASSWORD=.+' .env | grep -v 'your-secure-password-here' \
  && grep -E '^SCHEMA_SERVICE_KEY=.+' .env | grep -v 'dev-secret-key-change-in-production' \
  && echo OK
```

---

## 4. Start the stack

From `docker/`:

```bash
./start.sh
```

This runs `docker-compose --profile local up -d --build`. First build pulls base images and compiles the schema service (downloads ML models on first boot — allow ~5 minutes). Subsequent boots are seconds.

If `./start.sh` is not executable: `chmod +x start.sh stop.sh && ./start.sh`.

If `docker-compose` (v1) is missing but `docker compose` (v2) is present, run directly:

```bash
docker compose --profile local up -d --build
```

---

## 5. Wait for health and verify

The schema service has a 120s `start_period` for first-boot model download. Poll until all containers are healthy:

```bash
until [ "$(docker ps --filter 'name=dashboard-' --format '{{.Status}}' | grep -c '(healthy)')" = "5" ]; do
  docker ps --filter 'name=dashboard-' --format 'table {{.Names}}\t{{.Status}}'
  sleep 10
done
echo "all healthy"
```

Expected healthy containers: `dashboard-postgres`, `dashboard-trino`, `dashboard-schema-service`, `dashboard-nexus`, `dashboard-frontend`.

Endpoint smoke tests:

```bash
curl -fsS http://localhost:8000/health         # Nexus API
curl -fsS http://localhost:8001/health         # Schema Service
curl -fsS http://localhost:8081/v1/info        # Trino
curl -fsSI http://localhost:3000 | head -1     # Frontend (200 OK)
```

All four must return success. If any fail, jump to Troubleshooting.

---

## 6. Service map

| Service | URL | Purpose |
|---|---|---|
| Frontend | http://localhost:3000 | React UI |
| Nexus API | http://localhost:8000 | FastAPI backend (all frontend traffic proxies here) |
| Schema Service | http://localhost:8001 | FAISS table recommendations |
| Trino | http://localhost:8081 | Federated SQL engine |
| PostgreSQL | localhost:5432 | App + demo data; user `nexus_service`, db `dashboard` |

---

## 7. Stop / reset

```bash
./stop.sh           # stop containers, keep data volumes
./stop.sh --clean   # stop and wipe all data (destructive — confirm with user first)
```

Restart from a clean slate (data lost):

```bash
./stop.sh --clean && ./start.sh
```

---

## 8. Optional: prediction workers

Predictive intelligence workers are gated behind the `predictions` profile (and `predictions-timeseries` for the heavy AutoGluon image). Only start when explicitly needed:

```bash
docker compose --profile local --profile predictions up -d --build
```

Adds `prediction-worker-xgboost` (8401) and `prediction-worker-lightgbm` (8402). AutoGluon (`predictions-timeseries`) pulls a ~3.5 GB image — start only on request.

---

## 9. Optional: external PostgreSQL / Trino


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [actyze/dashboard](https://github.com/actyze/dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
