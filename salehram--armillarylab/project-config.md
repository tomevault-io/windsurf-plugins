---
trigger: always_on
description: >-
---


# ArmillaryLab — Project Reference Cache

> **Last updated:** 2026-05-22
> **App version:** 2.2.0 (`APP_VERSION` in `app.py`)
> **Purpose:** Save agent discovery time and token cost. Read this FIRST, explore SECOND.
> **Maintenance:** MANDATORY — before committing, if you modified app.py (models/routes/helpers/database schema), any template, astro_utils.py, **conditions_utils.py**, **calibration_utils.py**, time_utils.py, nina_integration.py, config/*.py, requirements.txt, or .env.example in a way that changes the project's structure or API surface (new/removed/renamed models, columns, routes, templates, env vars, dependencies), you MUST update the relevant section(s) below and bump the "Last updated" date. A stop-hook also enforces this as a safety net.

---

## 1. Tech Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Backend | Flask 3.x | Single-module app (`app.py`) |
| ORM | SQLAlchemy 2.x + Flask-SQLAlchemy 3.x | All models in `app.py` |
| Database | SQLite (dev) / PostgreSQL (prod) | Configured via `DATABASE_URL` env var |
| Migrations | Custom `config/migration.py` | Full bidirectional SQLite↔PostgreSQL migrator (not Alembic) |
| Astronomy | astropy 6.x + astroplan 0.10+ | Window/altitude + moon phase (Night Conditions) |
| External APIs | Open-Meteo, 7Timer | Weather & seeing (no API keys; server-side fetch in `conditions_utils.py`) |
| Templates | Jinja2 (Flask built-in) | Dark-themed Bootstrap 5 |
| Charts | Chart.js (vendored UMD) | Altitude-over-night plot |
| Frontend | Bootstrap 5 + Bootstrap Icons | No custom CSS/JS files — all inline |
| Server | gunicorn (prod), Flask dev server (dev) | Port 5000 |
| Container | Docker + docker-compose | Single `web` service |

---

## 2. Directory Structure

```
astroplanner/
├── app.py                    # Flask app: ALL models + ALL routes + helpers
├── astro_utils.py            # compute_target_window() — core astronomy
├── conditions_utils.py       # Night conditions: moon phase, weather, seeing, channel suggestion
├── calibration_utils.py      # Calibration frame tracking: suggestions, status, export totals
├── time_utils.py             # utc_to_local() + template filter registration
├── nina_integration.py       # NINA XML sequence + filter profile parsing
├── cli.py                    # Dev server entry: python cli.py
├── config/
│   ├── database.py           # configure_app() — DB URI + engine options
│   └── migration.py          # DatabaseMigrator class — bidirectional SQLite↔PostgreSQL
├── templates/                # 17+ Jinja2 templates (see §6)
├── static/
│   ├── css/                  # bootstrap.min.css, bootstrap-icons.min.css
│   ├── js/                   # bootstrap.bundle.min.js, chart.umd.js
│   ├── fonts/                # bootstrap-icons.woff, .woff2
│   └── images/               # armillarylab-logo.png, favicon.ico
├── tests/
│   ├── __init__.py
│   └── test_app.py           # pytest suite (~15 tests)
├── uploads/                  # User-uploaded final images (gitignored)
├── cache/                    # Weather/seeing forecast cache (gitignored)
│   └── conditions/           # JSON cache files keyed by lat/lon
├── requirements.txt          # 11 dependencies
├── .env / .env.example       # Observer config, DB URL, secrets
├── Dockerfile                # python:3.12-slim, gunicorn
├── docker-compose.yml        # Single web service, volume mounts
├── run_tests.py              # pytest runner script
├── armillarylab.db           # Bundled SQLite showcase data (tracked; omit via delete + flask init-db for blank)
└── README.md
```

---

## 3. Database Models (all in `app.py`)

> Models define the database schema. SQLAlchemy columns map directly to DB columns.
> Schema changes happen by modifying model classes, then `config/migration.py` handles syncing.

### GlobalConfig (`global_config`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| observer_lat | Float, default 24.7136 | Observer latitude |
| observer_lon | Float, default 46.6753 | Observer longitude |
| observer_elev_m | Float, default 600 | Elevation in meters |
| default_packup_time | String(5), default "01:00" | Default pack-up HH:MM |
| default_min_altitude | Float, default 30.0 | Default min altitude degrees |
| default_calibration_darks | Integer, default 0 | Default dark count per target |
| default_calibration_flats_per_channel | Integer, default 0 | Flats per plan channel |
| default_calibration_dark_flats_per_channel | Integer, default 0 | Dark flats per channel |
| default_calibration_bias | Integer, default 0 | Optional bias count |
| default_calibration_two_point | Boolean, default True | Split flats at midpoint + end |
| max_cloud_cover_pct | Integer, default 25 | Cloud cover % threshold for go/skip advice |
| timezone_name | String(64), default "Asia/Riyadh" | IANA timezone |
| updated_at | DateTime, default utcnow | |

Singleton row — `get_global_config()` creates one if missing. Replaces .env for observer settings in DB-backed config.

### TargetType (`target_types`)
| Column | Type | Notes |
|--------|------|-------|
| id | Integer PK | |
| name | String(64), unique, not null | "emission", "galaxy", "planetary", etc. |
| recommended_palette | String(16), not null | "SHO", "LRGB", etc. |
| description | Text | Why this palette works for this type |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [salehram/armillarylab](https://github.com/salehram/armillarylab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
