---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Important Constraints

**ABSOLUTELY FORBIDDEN COMMANDS:**
- **NEVER** run `git commit` or `git push` on main. ALWAYS open PRs.
- **NEVER** perform `ssh` or `scp` operations on the server without the user giving you explicit permissions
- The user will manually handle all git operations and file transfers to the server

## Working Environment

**Local Development:** Commands are run locally on the developer's machine. To execute commands on the production server, use `ssh revel "cd infra && <command>"` format.

**Server Directory:** The infrastructure is deployed in the `infra` directory on the production server.

## Repository Overview

This is the infrastructure repository for **Revel**, a Django-based application platform. It contains the complete Docker Compose setup orchestrating all application services, infrastructure components, and a comprehensive observability stack. This repository is part of a multi-repo architecture alongside:

- **revel-backend** - Django REST API with business logic, living at `../revel-backend`
- **revel-frontend** - SvelteKit web application, living at `../revel-frontend`
- **infra** (this repo) - Deployment and infrastructure configuration

The application runs on a **Hetzner CCX33** instance (8 vCPU, 32GB RAM, 240GB disk).

## Architecture

### Service Categories

**Application Services:**
- `web` - Django app running on Gunicorn (6 workers, 4 threads, gthread worker class)
- `frontend` - SvelteKit application on port 3000
- `celery_default` - Background task worker (4 concurrency)
- `beat` - Celery scheduler with Django database scheduler
- `flower` - Celery monitoring UI with Google SSO auth
- `telegram` - Telegram bot service

**Infrastructure Services:**
- `caddy` - Reverse proxy with automatic HTTPS (serves 4 domains)
- `revel_postgres` - PostGIS 17-3.5 with optimized configuration for 32GB RAM
- `pgbouncer` - Connection pooler (transaction mode, max 1000 client connections, pool size 25)
- `redis` - Cache and message broker (512MB maxmemory, LRU eviction, AOF persistence)

**Observability Stack:**
- `prometheus` - Metrics collection (30d retention)
- `alertmanager` - Alert routing with Pushover integration
- `loki` - Log aggregation
- `tempo` - Distributed tracing (OTLP on ports 4317/4318)
- `pyroscope` - Continuous profiling
- `alloy` - eBPF-based profiling collector (requires privileged mode)
- `grafana` - Visualization dashboard
- `postgres-exporter` - PostgreSQL metrics
- `redis-exporter` - Redis metrics
- `blackbox-exporter` - Health check probing

**Security:**
- `clamav` - Antivirus scanning (256MB max file size)

### Networking

All services run on `revel_network` (bridge network). Services communicate using container names as hostnames.

### Volume Management

Persistent data volumes:
- `revel_postgres_data` - Database (most critical)
- `redis_data` - Cache persistence
- `caddy_data` - SSL certificates
- `prometheus_data`, `loki_data`, `tempo_data` - Observability data
- `grafana_data` - Dashboard configurations

Bind mounts:
- `./media` - User uploads (shared between web, celery, telegram)
- `./geo-data` - Geographic data files
- `./sentinel` - LLM sentinel data

## Common Commands

### Starting and Stopping

```bash
# Start all services
docker compose up -d

# Start with deploy script (includes validation)
./deploy.sh up

# Stop all services
docker compose down

# Update to latest images
./deploy.sh update
# or manually:
docker compose pull
docker compose up -d
```

### Monitoring and Debugging

```bash
# View all service status
docker compose ps

# View logs for specific service
docker compose logs -f web
docker compose logs -f celery_default

# View logs with timestamps
docker compose logs -f --timestamps web

# Follow logs for multiple services
docker compose logs -f web celery_default

# Check health of all services
docker compose ps --format json | jq '.[] | {name: .Name, health: .Health}'
```

### Database Operations

```bash
# Access PostgreSQL directly (bypassing PgBouncer)
docker compose exec revel_postgres psql -U $DB_USER -d $DB_NAME

# Access via PgBouncer
docker compose exec pgbouncer psql -h localhost -p 6432 -U $DB_USER -d $DB_NAME

# Create backup
./deploy.sh backup
# or manually:
docker compose exec -T revel_postgres pg_dump -U $DB_USER $DB_NAME > backup_$(date +%Y%m%d_%H%M%S).sql

# Restore from backup
cat backup.sql | docker compose exec -T revel_postgres psql -U $DB_USER -d $DB_NAME
```

### Celery Operations

```bash
# Scale workers
docker compose up -d --scale celery_default=4

# Inspect active tasks
docker compose exec celery_default celery -A revel inspect active

# Purge all tasks from queue
docker compose exec celery_default celery -A revel purge
```

### Configuration Reloads

```bash
# Reload Prometheus configuration (without restart)
docker compose exec prometheus kill -HUP 1

# Reload Alertmanager configuration
docker compose restart alertmanager

# Reload Caddy configuration
docker compose exec caddy caddy reload --config /etc/caddy/Caddyfile
```

## Environment Configuration

The `.env` file controls all configuration. Critical variables:

**Database:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [letsrevel/infra](https://github.com/letsrevel/infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
