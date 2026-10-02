---
trigger: always_on
description: Use when serving the built SPA from the API (`DOCKFIN_WEB_DIR`), e.g. VPS smoke at `/opt/dockfin-smoke`.
---

# AGENTS.md — Dockfin development guide

Practical guide for humans and AI agents working on this repo. Prefer these commands over inventing new workflows.

## Layout (what matters day-to-day)

| Path | Role |
|------|------|
| `cmd/dockfin` | API binary (`serve`, `migrate`, `version`) |
| `internal/` | Go packages (httpapi, services, deploy, proxy, store, …) |
| `apps/web/` | React + Vite dashboard |
| `templates/compose/` | One-click service YAML catalog |
| `migrations/` | Goose SQL (run via `dockfin migrate`) |
| `deploy/docker/Dockerfile.api` | Single production image (API + baked Vite UI) |
| `deploy/compose/docker-compose.dev.yml` | Local Postgres + Redis (hot-reload mode) |
| `scripts/install.sh` | **Production** install — pulls `ghcr.io/foisalislambd/dockfin` |
| `scripts/install-dev.sh` | **Dev** install — builds `dockfin:local`, no registry pull |
| `.env` | Runtime config for hot-reload mode (copy from `.env.example`) |

Control-plane data:

| Mode | Where |
|------|-------|
| Hot reload (`go run`) | `DOCKFIN_DATA_DIR` (default `./data`) |
| Dev Docker (`install-dev.sh`) | Compose project `/data/dockfin` + volumes `dockfin-pg` / `dockfin-data` |
| Production (`install.sh`) | Same dir/volumes; image switched to GHCR |

**Important:** App/DB/service files on a *target server* are always under host path **`/data/dockfin/{applications,databases,services,proxy,backups}`** (hardcoded in Go over SSH). That is separate from the control-plane container’s `/data` volume.

---

## Install scripts (pick one)

| Script | When | Image | Dir |
|--------|------|-------|-----|
| `scripts/install.sh` | Real VPS / production | `ghcr.io/foisalislambd/dockfin:latest` (pull) | `/data/dockfin` |
| `scripts/install-dev.sh` | Agent/smoke testing full stack in Docker | `dockfin:local` (build from this repo) | `/data/dockfin` (same) |

Both scripts share `/data/dockfin` and the same Docker volumes so switching prod ↔ local keeps the DB. Do **not** run two stacks on port 8000 at once.

### Production (GHCR)

```bash
curl -fsSL https://raw.githubusercontent.com/foisalislambd/dockfin/main/scripts/install.sh | sudo bash

# Pin a version
sudo DOCKFIN_VERSION=1.0.9 bash -c 'curl -fsSL …/install.sh | bash'

# Update later
cd /data/dockfin && sudo docker compose pull && sudo docker compose up -d
```

Dashboard: `http://SERVER_IP:8000/` (ports 80/443 left free for Traefik).

### Development Docker stack (preferred for “run the whole product” on a VPS)

From the **repo root** (needs Docker + root for `/data`):

```bash
cd /root/dockfin   # or your clone path
sudo bash scripts/install-dev.sh
```

What it does:

1. `docker build -f deploy/docker/Dockerfile.api -t dockfin:local .`
2. Writes `/data/dockfin/docker-compose.yml` + `.env` (`DOCKFIN_ENV=development`)
3. `docker compose up -d --pull never --force-recreate` (Postgres + Dockfin on **:8000**)

```bash
# Health / version
curl -s http://127.0.0.1:8000/health
curl -s http://127.0.0.1:8000/api/v1/version

# Logs / restart
cd /data/dockfin && docker compose logs -f dockfin
cd /data/dockfin && docker compose restart dockfin
```

**After Go or UI code changes**, rebuild + recreate (same script is fine):

```bash
cd /root/dockfin && sudo bash scripts/install-dev.sh
```

Or manually:

```bash
cd /root/dockfin
docker build -f deploy/docker/Dockerfile.api --build-arg VERSION=dev -t dockfin:local .
cd /data/dockfin
unset DOCKFIN_DATABASE_URL DOCKFIN_MASTER_KEY DOCKFIN_HTTP_ADDR DOCKFIN_PUBLIC_URL 2>/dev/null || true
docker compose up -d --pull never --force-recreate dockfin
```

Do **not** export host `DOCKFIN_DATABASE_URL=…@127.0.0.1` when using compose — it overrides the container DB host (`postgres`). Secrets stay in `/data/dockfin/.env` only.

Optional overrides for `install-dev.sh`:

| Env | Default |
|-----|---------|
| `DOCKFIN_DIR` | `/data/dockfin` |
| `DOCKFIN_IMAGE` | `dockfin:local` |
| `DOCKFIN_VERSION` | `dev` (build-arg / version string) |
| `DOCKFIN_HOST_PORT` | `8000` |

---

## Prerequisites (hot-reload mode)

- Go **1.26+**
- Node.js **22+**
- Docker (for Postgres/Redis and deploying containers)

```bash
cp .env.example .env
# Set DOCKFIN_MASTER_KEY to ≥32 characters
```

---

## A) Everyday local development (hot reload UI)

Use this when iterating on API/UI quickly. For a full Docker control plane like production, use **`install-dev.sh`** instead.

### 1. Database

```bash
docker compose -f deploy/compose/docker-compose.dev.yml up -d
```

### 2. Migrate + API

```bash
set -a && source .env && set +a
go run ./cmd/dockfin migrate
go run ./cmd/dockfin serve
# or: make migrate && make api
```

API: `http://127.0.0.1:8000` · health: `/health` · version: `/api/v1/version`

Optional for templates + static UI when not using Vite:

```bash
export DOCKFIN_TEMPLATES_DIR="$PWD/templates/compose"
export DOCKFIN_WEB_DIR="$PWD/apps/web/dist"
```

### 3. Web (Vite)

```bash
cd apps/web
npm install   # first time
npm run dev   # http://localhost:5173 — proxies/CORS via DOCKFIN_CORS_ORIGINS
```

Use this mode when iterating on UI. API changes still need restarting `go run` / the binary.

---

## B) Rebuild + restart (binary + static UI)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [foisalislambd/dockfin](https://github.com/foisalislambd/dockfin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
