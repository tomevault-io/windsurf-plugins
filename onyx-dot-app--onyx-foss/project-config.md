---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# PROJECT KNOWLEDGE BASE

This file provides guidance to AI agents when working with code in this repository.

## KEY NOTES

- If you run into any missing python dependency errors, try running your command with `source .venv/bin/activate` \
  to assume the python venv.
- To make tests work, check the `.env` file at the root of the project to find an OpenAI key.
- If using `playwright` to explore the frontend, you can usually log in with username `a@example.com` and password
  `a`. The app can be accessed at `http://localhost:3000`.
- You should assume that all Onyx services are running. To verify, you can check the `backend/log` directory to
  make sure we see logs coming out from the relevant service.
- To connect to the Postgres database, use: `docker exec -it onyx-relational_db-1 psql -U postgres -c "<SQL>"`
- When making calls to the backend, always go through the frontend. E.g. make a call to `http://localhost:3000/api/persona` not `http://localhost:8080/api/persona`
- Put ALL db operations under the `backend/onyx/db` / `backend/ee/onyx/db` directories. Don't run queries
  outside of those directories.

## Project Overview

**Onyx** (formerly Danswer) is an open-source Gen-AI and Enterprise Search platform that connects to company documents, apps, and people. It features a modular architecture with both Community Edition (MIT licensed) and Enterprise Edition offerings.

### Background Workers (Celery)

Onyx uses Celery for asynchronous task processing with multiple specialized workers:

#### Worker Types

1. **Primary Worker** (`celery_app.py`)
   - Coordinates core background tasks and system-wide operations
   - Handles connector management, document sync, pruning, and periodic checks
   - Runs with 4 threads concurrency
   - Tasks: connector deletion, vespa sync, pruning, LLM model updates, user file sync

2. **Docfetching Worker** (`docfetching`)
   - Fetches documents from external data sources (connectors)
   - Spawns docprocessing tasks for each document batch
   - Implements watchdog monitoring for stuck connectors
   - Configurable concurrency (default from env)

3. **Docprocessing Worker** (`docprocessing`)
   - Processes fetched documents through the indexing pipeline:
     - Upserts documents to PostgreSQL
     - Chunks documents and adds contextual information
     - Embeds chunks via model server
     - Writes chunks to Vespa vector database
     - Updates document metadata
   - Configurable concurrency (default from env)

4. **Light Worker** (`light`)
   - Handles lightweight, fast operations
   - Tasks: vespa metadata sync, connector deletion, doc permissions upsert, checkpoint cleanup, index attempt cleanup
   - Higher concurrency for quick tasks

5. **Heavy Worker** (`heavy`)
   - Handles resource-intensive operations
   - Tasks: connector pruning, document permissions sync, external group sync, CSV generation
   - Runs with 4 threads concurrency

6. **KG Processing Worker** (`kg_processing`)
   - Handles Knowledge Graph processing and clustering
   - Builds relationships between documents
   - Runs clustering algorithms
   - Configurable concurrency

7. **Monitoring Worker** (`monitoring`)
   - System health monitoring and metrics collection
   - Monitors Celery queues, process memory, and system status
   - Single thread (monitoring doesn't need parallelism)
   - Cloud-specific monitoring tasks

8. **User File Processing Worker** (`user_file_processing`)
   - Processes user-uploaded files
   - Handles user file indexing and project synchronization
   - Configurable concurrency

9. **Beat Worker** (`beat`)
   - Celery's scheduler for periodic tasks
   - Uses DynamicTenantScheduler for multi-tenant support
   - Schedules tasks like:
     - Indexing checks (every 15 seconds)
     - Connector deletion checks (every 20 seconds)
     - Vespa sync checks (every 20 seconds)
     - Pruning checks (every 20 seconds)
     - KG processing (every 60 seconds)
     - Monitoring tasks (every 5 minutes)
     - Cleanup tasks (hourly)

#### Key Features

- **Thread-based Workers**: All workers use thread pools (not processes) for stability
- **Tenant Awareness**: Multi-tenant support with per-tenant task isolation. There is a
  middleware layer that automatically finds the appropriate tenant ID when sending tasks
  via Celery Beat.
- **Task Prioritization**: High, Medium, Low priority queues
- **Monitoring**: Built-in heartbeat and liveness checking
- **Failure Handling**: Automatic retry and failure recovery mechanisms
- **Redis Coordination**: Inter-process communication via Redis
- **PostgreSQL State**: Task state and metadata stored in PostgreSQL

#### Important Notes

**Defining Tasks**:

- Always use `@shared_task` rather than `@celery_app`
- Put tasks under `background/celery/tasks/` or `ee/background/celery/tasks`
- Never enqueue a task without an expiration. Always supply `expires=` when
  sending tasks, either from the beat schedule or directly from another task. It
  should never be acceptable to submit code which enqueues tasks without an
  expiration, as doing so can lead to unbounded task queue growth.

**Defining APIs**:
When creating new FastAPI APIs, do NOT use the `response_model` field. Instead, just type the
function.

**Testing Updates**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onyx-dot-app/onyx-foss](https://github.com/onyx-dot-app/onyx-foss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
