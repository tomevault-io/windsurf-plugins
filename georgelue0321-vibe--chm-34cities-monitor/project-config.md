---
trigger: always_on
description: A modular Python data pipeline that:
---

# AGENTS.md — China Housing Monitor (CHM)

## What this is

A modular Python data pipeline that:
1. Manages a local SQLite DB (`china_monitor_db.sqlite`) with 8 tables of Chinese housing market data for 34 cities (v0.8, expanded from 18)
2. Scrapes Lianjia for real-time listing counts and prices, with a mathematically-documented fallback when blocked
3. Computes a multi-factor "Bottom Signal Score" (0-100) per city per month
4. Compiles everything into a standalone single-file HTML SPA (`chm.html`) with all data injected as `window.MONITOR_DB`

No dependencies beyond Python 3 standard library. No package.json, no requirements.txt, no venv.

## Prerequisites

**Core pipeline** (init, score, compile HTML):
- Python 3.11+
- No external dependencies

**Data collection** (optional, for real-time updates):
- [browser-use](https://github.com/browser-use/browser-use) CLI: `pip install "browser-use[core]"`
- Chrome/Chromium with remote debugging enabled
- **Must use `--headed` mode** for all browser-use commands (百度、央行、中指研究院会拦截 headless)

```bash
# Verify browser-use installation
browser-use doctor

# Test headed mode
browser-use --headed open "https://www.baidu.com"
```

**Codex sandbox note**: `browser-use` may abort inside the Codex shell sandbox while
probing macOS display size via AppKit (`NSScreen.mainScreen`). If `browser-use doctor`
exits with code 134 in sandbox, rerun `browser-use` commands outside the sandbox with
explicit approval. Do not diagnose this as a Chrome/百度 problem until
`browser-use doctor` passes outside the sandbox.

For detailed setup, see `skills/storage-event-scanner/SKILL.md`.

## Commands

```bash
# Run the full pipeline: init DB, scrape 34 cities, compute scores, compile HTML
python3 -m china_housing_monitor

# Skip scraping, just regenerate HTML from existing DB
python3 -m china_housing_monitor --no-scrape

# Only initialize/seed the database
python3 -m china_housing_monitor --init-only

# Run the verification test suite (31 tests, uses a copied test DB — safe for production)
python3 tests/test_scoring_rigor.py

# Start local HTTP server for preview
python3 -m http.server 8080
```

## Architecture

```
china_housing_monitor/                  ← Python package (v1.0.0)
├── __init__.py                         ← Package init, version
├── __main__.py                         ← CLI entry point (argparse)
├── compat.py                           ← Backward compatibility shim for tests
├── config.py                           ← Constants, paths, city definitions, scoring params
├── crawler.py                          ← Lianjia scraper with SSL bypass + fallback chain
├── db/
│   ├── __init__.py
│   ├── init.py                         ← init_db(), backup_db(), add_column_if_not_exists()
│   └── seed.py                         ← seed_historical_data() — NBS, transaction, storage, opinions
├── scoring/
│   ├── __init__.py
│   ├── factors.py                      ← Scoring helpers: calc_s_price, storage_recency, PBOC mapping
│   └── bottom.py                       ← compute_bottom_score, generate_warnings, decide_city_status_timeline
├── data/
│   ├── __init__.py
│   ├── payload.py                      ← fetch_data_payload() — assembles full JSON payload from SQLite
│   └── charts.py                       ← Chart helpers: visibility, evidence grade, signal interpretation
└── report/
    ├── __init__.py
    ├── generator.py                    ← generate_html_report() — reads templates, injects data
    ├── static/
    │   ├── style.css                   ← Custom CSS (scrollbars, transitions, nav)
    │   ├── nav.js                      ← Navigation: toggleNavMobile, onCityChange, updateNavButtons
    │   ├── gauge.js                    ← renderNationalGauge — PBOC radial bar charts
    │   ├── rankings.js                 ← renderRankings — city ranking list
    │   ├── charts.js                   ← NBS index, transaction, score history, scraped charts
    │   └── dashboard.js                ← renderCityDashboard — main city detail view
    └── templates/
        ├── base.html                   ← HTML skeleton, footer, script tags
        ├── header.html                 ← Header/navbar with city tier navigation
        ├── left_column.html            ← Storage events + rankings
        └── right_column.html           ← City details, scoring factors, signal interpretation

china_monitor_db.sqlite                 ← The database (gitignored)
chm.html                                ← Generated output, open directly in browser
scratch/verify_scoring_rigor.py         ← Test suite (31 tests)
```

## Module dependency graph

```
__main__.py
    ├── config
    ├── db.init
    ├── crawler
    └── report.generator
            ├── config
            ├── data.payload
            │       ├── config
            │       ├── scoring.factors
            │       ├── scoring.bottom
            │       └── data.charts
            └── (reads templates/ + static/ files)

compat.py (test shim)
    ├── config
    ├── db.init
    ├── db.seed
    ├── scoring.factors
    ├── scoring.bottom
    ├── crawler
    ├── data.payload
    ├── data.charts
    └── report.generator
```

## Critical gotchas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [georgelue0321-vibe/chm_34cities_monitor](https://github.com/georgelue0321-vibe/chm_34cities_monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
