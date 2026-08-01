---
trigger: always_on
description: This file provides guidance to WARP (warp.dev) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Databús is a distributed transit data system implementing GTFS Schedule and GTFS Realtime specifications. The system consists of multiple services coordinated via message brokers, with Django backend as the control plane and separate Python services for real-time processing and feed generation.

**Tech Stack:** Django 5.2+, Python 3.11+, PostgreSQL/PostGIS, Redis, RabbitMQ, MQTT, Celery, Docker

## Development Commands

### Initial Setup

```bash
# Docker-based development (recommended)
./scripts/dev.sh

# Non-Docker setup
python -m venv .venv
source .venv/bin/activate  # On macOS/Linux
uv pip install -r backend/requirements.txt
cp .env.example .env  # Configure environment variables
cd backend && python manage.py migrate
```

### Running Services

**Docker (recommended):**

```bash
./scripts/dev.sh  # Starts all services
docker compose -f compose.dev.yml logs -f  # View logs
docker compose -f compose.dev.yml logs -f orchestrator  # Single service logs
docker compose -f compose.dev.yml down  # Stop all services
```

**Non-Docker (requires running services separately in multiple terminals):**

```bash
# Terminal 1: Django
cd backend && python manage.py runserver

# Terminal 2: Redis
redis-server

# Terminal 3: RabbitMQ
# (see installation docs for your OS)

# Terminal 4: Publisher (Celery worker)
cd publisher && uv run python -m celery -A publisher worker -l info

# Terminal 5: Scheduler (Celery beat)
cd scheduler && uv run python -m celery -A scheduler beat -l info
```

### Database Operations

```bash
# Docker
docker compose -f compose.dev.yml exec orchestrator uv run python manage.py makemigrations
docker compose -f compose.dev.yml exec orchestrator uv run python manage.py migrate
docker compose -f compose.dev.yml exec orchestrator uv run python manage.py shell
docker compose -f compose.dev.yml exec orchestrator uv run python manage.py createsuperuser

# Custom management command to refresh GTFS model FKs
docker compose -f compose.dev.yml exec orchestrator uv run python manage.py update_foreign_keys

# Load fixture data (bUCR GTFS)
docker compose -f compose.dev.yml exec orchestrator uv run python manage.py loaddata gtfs.json

# Non-Docker
cd backend
python manage.py makemigrations
python manage.py migrate
python manage.py shell
```

### Code Quality

```bash
# Run from backend/ directory
cd backend

# Linting and formatting
ruff check .
ruff format .

# Type checking
mypy .

# Tests (minimal coverage currently)
pytest
pytest tests/ -v
pytest tests/test_specific.py::test_function  # Single test
```

### Accessing Services

- Orchestrator: http://localhost:8000
- Django Admin: http://localhost:8000/admin
- API Root: http://localhost:8000/api/
- API Docs: http://localhost:8000/api/docs/
- RabbitMQ Management: http://localhost:15672 (guest/guest)
- Prefect Analytics: http://localhost:4200

## Architecture

### Service-Oriented Architecture

The system is composed of independent services communicating asynchronously:

1. **orchestrator** (Django) - Control plane and HTTP API
   - Django apps: `gtfs` (submodule), `feed`, `api`, `website`
   - Manages domain models, issues commands, exposes REST APIs
   - Does NOT process real-time telemetry or maintain operational state
   - Located in: `backend/`

2. **realtime-engine** (Python) - Real-time processing
   - Consumes MQTT telemetry and AMQP commands
   - Updates authoritative state in Redis
   - Emits observations to message broker
   - Located in: `realtime-engine/`

3. **publisher** (Celery worker) - GTFS Realtime generation
   - Reads state snapshots from Redis
   - Generates protobuf feeds (`vehicle_positions.pb`, `trip_updates.pb`)
   - Emits assertions to message broker
   - Located in: `publisher/`

4. **scheduler** (Celery beat) - Temporal orchestration
   - Triggers periodic publishing tasks
   - Located in: `scheduler/`

5. **analytics-engine** (Prefect) - Batch processing and ML
   - Processes historical data for insights
   - Located in: `analytics-engine/`

### Infrastructure Services

- **database** - PostgreSQL with PostGIS (durable persistence)
- **state** - Redis (authoritative in-memory operational state)
- **message-broker** - RabbitMQ (AMQP for commands/observations/assertions)
- **telemetry-broker** - NanoMQ (telemetry ingestion from vehicles)

### Key Architectural Principles

- **Single writer per responsibility** - Each service owns specific concerns
- **Async-first** - Services communicate via brokers, not synchronous calls
- **In-memory state is authoritative for real-time** - Database is NOT used for coordination
- **Explicit message semantics** - Commands (orchestrator→engine), observations (engine→orchestrator), assertions (publisher→orchestrator)

### Data Flow Example

1. Dispatcher issues "begin run" command via backend HTTP API
2. Backend stores run metadata in PostgreSQL and emits command to RabbitMQ
3. Realtime-engine receives command, initializes run state in Redis
4. Vehicle sends telemetry via MQTT
5. Realtime-engine processes telemetry, updates Redis state, emits observations
6. Scheduler triggers publisher task (every 15 seconds)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simovilab/databus](https://github.com/simovilab/databus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
