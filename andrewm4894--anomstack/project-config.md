---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Anomstack is an open-source anomaly detection system built on Dagster and FastHTML. It provides ML-powered anomaly detection for metrics from various data sources (BigQuery, Snowflake, ClickHouse, DuckDB, SQLite, etc.) with built-in alerting via email/Slack.

## Development Commands

### Local Development
- `make local` - Start Dagster locally with dev setup
- `make dashboard` - Start FastHTML dashboard locally (port 5003)
- `make dashboard-uvicorn` - Start dashboard with uvicorn (hot reload)
- `make dashboard-local-dev` - Start dashboard with seeded test data
- `make dev` - Setup local development environment and install dependencies

### Docker Operations
- `make docker` - Start all services with Docker Compose
- `make docker-dev` - Start with local development images  
- `make docker-smart` - Build fresh images and start containers
- `make docker-build` - Build all images locally
- `make docker-logs` - View logs for all containers
- `make docker-logs-<service>` - View logs for specific service (code, dagit, dashboard, daemon)
- `make docker-shell-<service>` - Get shell access to running containers
- `make docker-restart` - Restart all containers (useful for .env changes)
- `make docker-stop` - Stop all containers

### Testing & Quality
- `pytest` or `make tests` - Run test suite (includes documentation link checking)
- `make test-examples` - Run only example ingest function tests
- `make coverage` - Run tests with coverage report
- `make pre-commit` - Run pre-commit hooks (ruff linting)

### Documentation Commands
- `make docs` or `make docs-start` - Start Docusaurus dev server with live reload
- `make docs-build` - Build static documentation site (includes broken link checking)
- `make docs-test` - Test documentation for broken links
- `make docs-serve` - Serve built documentation locally
- `make docs-clear` - Clear documentation build cache
- `make docs-install` - Install documentation dependencies

### Database Seeding & Examples
- `make seed-local-db` - Seed local DB with python_ingest_simple data
- `make seed-local-db-all` - Seed with all example metric batches
- `make seed-local-db-custom BATCHES='batch1,batch2' DB_PATH='path/to/db'`
- `make run-example EXAMPLE=<name>` - Test individual example ingest functions
- `make list-examples` - List all 26+ available examples

### Configuration & Hot Reload
- `make reload-config` - Reload configuration without restarting containers
- `make enable-auto-reload` - Enable automatic config reloading
- `make enable-config-watcher` - Enable smart config file watcher

### Reset & Cleanup Operations
- `make reset-interactive` - Interactive reset with guided options
- `make reset-gentle` - Rebuild containers (safest reset)
- `make reset-medium` - Remove containers, keep data volumes
- `make reset-nuclear` - Remove everything including local data
- `make reset-full-nuclear` - Nuclear + full docker system cleanup (maximum cleanup)
- `make dagster-cleanup-status` - Show current Dagster storage usage
- `make dagster-cleanup-minimal` - Remove old logs only (safe)
- `make dagster-cleanup-standard` - Clean up old Dagster runs (older than 30 days)
- `make dagster-cleanup-aggressive` - Remove runs older than 7 days
- `make kill-long-runs` - Manually kill any Dagster runs exceeding configured timeout

### Fly.io Deployment Commands
- `make fly-preview` - Preview environment variables that will be set as Fly secrets
- `make fly-deploy` - Deploy to Fly.io (reads .env file automatically)
- `make fly-deploy-demo` - Deploy with demo profile (enables demo metric batches)
- `make fly-deploy-production` - Deploy with production profile
- `make fly-deploy-development` - Deploy with development profile (all examples enabled)
- `make fly-deploy-demo-fresh` - Deploy with fresh build (clears Docker cache first)
- `make fly-build-test` - Test Fly.io build locally before deploying
- `make fly-docker-clean` - Clean Docker cache for Fly builds
- `make fly-cleanup` - Run disk cleanup on Fly instance (requires SSH access)
- `make fly-cleanup-preview` - Preview cleanup on Fly instance (dry run)
- `make fly-status` - Check Fly.io app status (requires FLY_APP env var)
- `make fly-logs` - View Fly.io app logs (requires FLY_APP env var)
- `make fly-ssh` - SSH into Fly.io app (requires FLY_APP env var)

## Architecture

### Container Architecture
Anomstack uses a simplified 3-container Docker architecture:
- **anomstack_webserver**: Consolidated Dagster webserver with embedded user code (no separate gRPC server)
- **anomstack_daemon**: Dagster daemon for job scheduling and execution
- **anomstack_dashboard**: FastHTML dashboard for metrics visualization

This consolidated approach eliminates the previous gRPC code server, reducing network overhead and improving reliability through direct Python module loading.

### Core Components
- **anomstack/**: Main application code
  - `main.py`: Dagster definitions and job orchestration
  - `config.py`: Configuration management
  - `jobs/`: Dagster jobs (ingest, train, score, alert, plot)
  - `ml/`: Machine learning components (PyOD models)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrewm4894/anomstack](https://github.com/andrewm4894/anomstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
