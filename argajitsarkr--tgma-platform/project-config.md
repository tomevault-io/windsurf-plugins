---
trigger: always_on
description: TGMA (Tripura Gut Microbiome in Adolescents) — Flask 3.1 research data management platform.
---

# CLAUDE.md — Project Intelligence for TGMA Platform

## Project Overview
TGMA (Tripura Gut Microbiome in Adolescents) — Flask 3.1 research data management platform.
ICMR-funded study at Tripura University. Self-hosted on Dell PowerEdge R730, LAN-only, Docker deployment.

**Status (April 2026)**: Platform is **feature-complete**. 65+ source files, 44 tests passing, Docker deployment ready. Features: KoboToolbox sync, participant edit/delete UI, demo-data wipe script, label-kit batched reprint (Seznik thermal printer), per-participant Document Vault (consent/assent/info-sheet/questionnaire/photos, PDF+JPG+PNG, role-gated upload/delete).

## Deployment
- **Server**: PowerEdge R730, Ubuntu, Docker Compose at `/home/mmilab/Desktop/tgma-platform`
- **Access (LAN)**: http://192.168.1.35:8100
- **Access (public)**: ngrok HTTPS tunnel — URL changes on restart; get current URL with `sudo docker compose logs ngrok | grep url=` or run `bash scripts/ngrok_url.sh` from the server
- **Database**: PostgreSQL 16 in Docker, volume `tgma-platform_pgdata`
- **Rebuild flow**: `git pull && sudo docker compose down && sudo docker compose up -d --build`
- **Re-seed / upsert users**: `sudo docker compose exec web python scripts/init_db.py`
- **Re-seed with synthetic data**: `sudo docker compose exec web python scripts/init_db.py --synthetic`
- **Wipe demo data (keeps users + schema)**: `sudo docker compose exec web python scripts/wipe_data.py --confirm` — deletes all participants (cascades), ID allocations, and KoboSync logs
- **Full reset (wipes data)**: add `-v` flag to `docker compose down`
- **Dev machine**: Windows (Anaconda), run with `conda run -n base python`
- **Ngrok setup (one-time)**: Sign up at ngrok.com, add `NGROK_AUTHTOKEN=<token>` to `.env` on server, then rebuild

## Build Verification
```bash
# App factory smoke test
conda run -n base python -c "from app import create_app; app = create_app('testing'); print('OK')"

# Full test suite (44 tests)
conda run -n base python -m pytest tests/ -v
```

## Complete File Inventory

### App Core
| File | Purpose |
|------|---------|
| `wsgi.py` | WSGI entry point |
| `config.py` | DevelopmentConfig / ProductionConfig / TestingConfig; study params (TARGET_ENROLLMENT=440, TARGET_SAMPLES_YEAR1=160, SEQUENCING_BATCH_SIZE=32); KOBO_API_URL config |
| `requirements.txt` | All Python deps (Flask 3.1, SQLAlchemy, pandas, python-barcode, gunicorn, etc.) |
| `.env.example` | Template for environment variables (DB, Kobo, upload paths) |
| `app/__init__.py` | App factory — registers all 10 blueprints (incl. kobo, documents), extensions, context processor |
| `app/extensions.py` | SQLAlchemy, Migrate, LoginManager, CSRFProtect |
| `app/auth.py` | Login/logout blueprint, Flask-Login user_loader |

### Models (8 files)
| File | Tables |
|------|--------|
| `app/models/__init__.py` | Re-exports all models |
| `app/models/user.py` | `users` — bcrypt password hashing, roles (pi, co_pi, bioinformatician, field_supervisor) |
| `app/models/participant.py` | `participants` — PK is `tracking_id` (VARCHAR 20), enrollment status, GPS coords |
| `app/models/clinical.py` | `health_screenings`, `anthropometrics`, `menstrual_data` — computed BMI, waist-hip ratio |
| `app/models/survey.py` | `lifestyle_data`, `environment_ses` — diet, activity, SES data from KoboToolbox |
| `app/models/sample.py` | `samples`, `sample_shipments` — freezer positions (UNIQUE constraint), chain of custody |
| `app/models/results.py` | `hormone_results`, `sequencing_results`, `id_allocations` — HOMA-IR, TG/HDL computed props |
| `app/models/admin.py` | `audit_log`, `blood_reports`, `kobo_sync_log`, `participant_documents` — PDF upload storage for diagnostics + sync run history + per-participant scanned forms/photos |

### Route Blueprints (10 files)
| File | URL Prefix | Key Features |
|------|-----------|--------------|
| `app/routes/__init__.py` | — | Package init |
| `app/routes/dashboard.py` | `/` | Stats cards, enrollment progress, district breakdown charts |
| `app/routes/participants.py` | `/participants` | CRUD, server-side DataTables API (`api_data`), detail, edit (POST), delete (POST) — edit/delete gated to PI/Co-PI/Bioinformatician; delete cascades to all child records |
| `app/routes/samples.py` | `/samples` | Register, tracker, freezer map, dispatch, detail; ALLOWED_SAMPLE_TYPES = ['stool', 'saliva_cortisol'] |
| `app/routes/diagnostics.py` | `/diagnostics` | Blood report PDF upload (NOT Excel import) |
| `app/routes/ids.py` | `/ids` | Bulk ID allocation per district; batches table with re-print, Excel kit, per-participant thermal print (50x30mm page / 25mm sticker) and delete buttons; `thermal_labels()` generates 10 QR code PNGs in-memory via `qrcode` + base64 data URIs; `delete_allocation()` and `delete_batch()` with participant-exists safety check; LABEL_KIT = 10 labels (blood vials removed) |
| `app/routes/quality.py` | `/quality` | GPS bounds check, outlier detection (|Z|>3), duplicates, missing data, incomplete sample sets |
| `app/routes/kobo.py` | `/kobo` | Manual "Sync Now" button, full re-sync, sync log history, per-run detail view. Restricted to PI/Co-PI/Bioinformatician. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/argajitsarkr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
