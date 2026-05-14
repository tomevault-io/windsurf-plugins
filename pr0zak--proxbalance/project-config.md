---
trigger: always_on
description: ProxBalance is an intelligent cluster monitoring and VM/CT migration management system for **Proxmox VE**. It provides real-time metrics, a penalty-based scoring algorithm for node health, AI-powered migration recommendations, automated migration scheduling, and an interactive web dashboard.
---

# CLAUDE.md - ProxBalance AI Assistant Guide

## Project Overview

ProxBalance is an intelligent cluster monitoring and VM/CT migration management system for **Proxmox VE**. It provides real-time metrics, a penalty-based scoring algorithm for node health, AI-powered migration recommendations, automated migration scheduling, and an interactive web dashboard.

**Target users**: Proxmox administrators managing multi-node clusters.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python 3.8+ (Flask 2.0+, Gunicorn) |
| Frontend | React 17+ (JSX), Tailwind CSS (CDN), Chart.js |
| Build Tool | esbuild (JSX bundling) |
| Reverse Proxy | Nginx |
| Process Manager | Systemd (services + timers) |
| Data Storage | SQLite (WAL mode) + JSON config files |
| Proxmox API | proxmoxer library |
| AI Providers | OpenAI, Anthropic Claude, Ollama |
| Notifications | Pushover, Email/SMTP, Telegram, Discord, Slack, Webhooks |

## Repository Structure

```
ProxBalance/
├── app.py                       # Flask entry point (~63 lines, modular Blueprint architecture)
├── collector_api.py             # Proxmox data collection service (834 lines)
├── ai_provider.py               # AI provider abstraction (OpenAI/Anthropic/Ollama)
├── notifications.py             # Multi-provider notification system
├── automigrate.py               # Automated migration orchestrator (~1,500 lines)
├── update_manager.py            # Update checking and branch management
├── update_timer.py              # Update timer helper
├── generate_recommendations.py  # Background recommendation generation
├── set_cluster_preset.py        # Cluster size preset configuration
│
├── proxbalance/                 # Core backend package (19 domain modules)
│   ├── __init__.py              # Package exports
│   ├── constants.py             # Shared path constants, file paths, tuning values
│   ├── config_manager.py        # Config loading/saving, Proxmox client (type-hinted)
│   ├── cache.py                 # In-memory cache with 60s TTL
│   ├── db.py                    # SQLite connection management, schema DDL, JSON migration
│   ├── migration_db.py          # Migration history, automation state, recommendation tracking
│   ├── error_handlers.py        # Centralized Flask error handling, @api_route decorator
│   ├── scoring.py               # Penalty-based scoring algorithm (~900 lines)
│   ├── recommendations.py       # Recommendation engine (~838 lines)
│   ├── recommendation_analysis.py # Confidence scoring, structured reasons, conflict detection
│   ├── storage.py               # Storage compatibility and verification
│   ├── distribution.py          # Guest distribution balancing across nodes
│   ├── migrations.py            # Migration execution logic (~643 lines)
│   ├── evacuation.py            # Node evacuation planning and sessions (~821 lines)
│   ├── forecasting.py           # Trend projection, forecast recommendations, score history
│   ├── metrics_store.py         # Node/guest metrics time-series storage (SQLite)
│   ├── guest_profiles.py        # Guest behavior profiling and classification (SQLite)
│   ├── patterns.py              # Workload pattern detection
│   ├── outcomes.py              # Migration outcome tracking (SQLite)
│   ├── trend_analysis.py        # Node/guest trend analysis
│   ├── execution_planner.py     # Topological execution ordering
│   ├── reporting.py             # Summaries, capacity advisories
│   ├── settings_mapper.py       # Simplified settings ↔ penalty config mapping
│   │
│   └── routes/                  # Flask Blueprints (all API endpoints, use @api_route)
│       ├── __init__.py          # register_blueprints() — registers all 10 blueprints
│       ├── analysis.py          # /api/cluster-analysis, /api/cluster-summary, etc.
│       ├── automation.py        # /api/automigrate/* endpoints
│       ├── config.py            # /api/config endpoints
│       ├── evacuation.py        # /api/nodes/evacuate endpoints
│       ├── guests.py            # /api/guests/* endpoints
│       ├── migrations.py        # /api/migrate endpoint
│       ├── notifications.py     # /api/notifications/test endpoint
│       ├── penalty.py           # /api/penalty-config endpoint
│       ├── recommendations.py   # /api/recommendations endpoints
│       └── system.py            # /api/update/*, /api/health, etc.
│
├── src/                         # Frontend source (React JSX, componentized)
│   ├── index.jsx                # Root component + hook composition (~560 lines)
│   ├── hooks/                   # 11 custom React hooks
│   │   ├── useDarkMode.js       # Dark mode toggle
│   │   ├── useAuth.js           # Permissions, token validation
│   │   ├── useUIState.js        # Page routing, collapsed sections, localStorage
│   │   ├── useConfig.js         # Config loading/saving, penalty config
│   │   ├── useEvacuation.js     # Maintenance nodes, evacuation state
│   │   ├── useUpdates.js        # System info, updates, branch management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pr0zak/ProxBalance](https://github.com/Pr0zak/ProxBalance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
