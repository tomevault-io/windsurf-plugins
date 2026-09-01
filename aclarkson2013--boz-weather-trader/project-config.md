---
trigger: always_on
description: Boz Weather Trader is a free, open-source automated trading bot for Kalshi weather prediction markets. It fetches weather forecast data from NWS and Open-Meteo, generates probability distributions for temperature outcomes, compares them to Kalshi market prices, and executes trades when it finds positive expected value (+EV).
---

# Boz Weather Trader — Project Guide

## What This Project Is

Boz Weather Trader is a free, open-source automated trading bot for Kalshi weather prediction markets. It fetches weather forecast data from NWS and Open-Meteo, generates probability distributions for temperature outcomes, compares them to Kalshi market prices, and executes trades when it finds positive expected value (+EV).

**PRD:** See `PRD.md` for the full product requirements document (v0.5+).

**Algo history & performance:** See `docs/ALGO_CHANGELOG.md` for the running history of every change to *how the bot predicts and decides trades* (prediction pipeline, probability model, EV/risk logic, order execution), each paired with its measured live-performance effect, plus dated performance reviews. **Read it before modifying prediction or trading logic, and before any performance review. Update it whenever a change alters prediction/probability/EV/sizing/risk/execution behavior, and append a snapshot whenever a performance review is run.**

## Architecture Overview

```
frontend/          → Next.js PWA (dashboard, onboarding, trade queue)
backend/
  ├── main.py      → FastAPI app entry point, /metrics endpoint, middleware stack, WebSocket lifespan
  ├── celery_app.py → Celery config, beat schedule, task signal instrumentation
  ├── weather/     → Agent 1: NWS + Open-Meteo data pipeline
  ├── kalshi/      → Agent 2: Kalshi API client (auth, orders, markets, WS feed, Redis cache)
  ├── prediction/  → Agent 3: Statistical ensemble + multi-model ML (XGBoost+RF+Ridge) + bracket probabilities + accuracy tracking + auto-retrain
  ├── trading/     → Agent 4: EV calculator (split YES/NO thresholds, realistic fee mode), Kelly sizing, risk controls, trade queue, bracket cap, guardrails (divergence cap, prob blending, YES floor), Kalshi-based settlement, resting order sync
  ├── backtesting/ → Backtesting engine: day-by-day simulation, synthetic prices, metrics
  ├── websocket/   → Real-time event push (Redis pub/sub → WebSocket → SWR revalidation)
  └── common/      → Shared schemas, config, database, logging, middleware, metrics
VERSION              → Single source of truth for app version (semver, read by backend + Docker + pyproject.toml)
scripts/updater/     → Self-update sidecar (Dockerfile, server.py, update.sh) — Docker socket mount for git pull + rebuild
monitoring/
  ├── prometheus/  → Prometheus scrape config + alerting rules
  │   ├── prometheus.yml   → Scrape config, rule_files, alertmanager target
  │   └── rules/           → 6 alert rule YAML files (18 rules across http, celery, trading, weather, targets, kalshi_ws)

  ├── alertmanager/        → Alertmanager config (webhook routing, severity-based repeat, inhibit rules)
  └── grafana/     → Grafana provisioning + dashboard JSON files
      ├── provisioning/  → Auto-provisioned datasources + dashboard provider
      └── dashboards/    → API Overview (8 panels) + Trading & Weather (10 panels) + Kalshi WS Feed (6 panels)
tests/                   → 1573 backend + 268 frontend = 1841 tests
  ├── common/      → Shared module tests: config, schemas, models, logging, encryption, middleware, metrics (123)
  ├── training/    → Training API endpoint tests (11)
  ├── weather/     → Weather pipeline: NWS, Open-Meteo, normalizer, stations, CLI parser, scheduler (140)
  ├── kalshi/      → Kalshi client: auth, REST, WS, markets, orders, models, cache, market feed (143)
  ├── prediction/  → Prediction engine: ensemble, multi-model ML, brackets, error dist, bias correction, accuracy, calibration, pipeline, source weights, retraining (280)
  ├── trading/     → Trading engine: EV calc (split YES/NO thresholds, fee modes) + guardrails, Kelly sizing, risk, cooldowns, queue, executor, scheduler, safety, sync, retraining trigger, bracket cap, resting order sync, orderbook pricing (389)
  ├── backtesting/ → Backtesting engine: schemas, risk sim, data loader, engine, metrics, integration (95)
  ├── api/         → API endpoints: auth, dashboard, dashboard stats, health, markets, queue, settings, trades, trades/sync, accuracy (incl. model edge), optimization, calendar, version, update, training (180)
  ├── websocket/   → WebSocket: events, manager, subscriber, router (40)
  ├── e2e/         → End-to-end smoke tests (35)
  ├── integration/ → Cross-module integration tests (47)
  ├── updater/     → Updater sidecar server tests (7)
  └── (root)       → Grafana dashboards, alert rules, alertmanager config validation (92)
```

## Tech Stack

- **Backend:** Python 3.11+, FastAPI, Celery + Redis, PostgreSQL
- **Frontend:** Next.js 14+, React, Tailwind CSS, PWA (Workbox)
- **ML/Stats:** scipy, numpy, XGBoost, scikit-learn (Gaussian CDF + multi-model ML ensemble: XGBoost + Random Forest + Ridge)
- **Monitoring:** prometheus-client, Prometheus, Grafana (auto-provisioned dashboards), Alertmanager (webhook alerts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aclarkson2013/boz-weather-trader](https://github.com/aclarkson2013/boz-weather-trader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
