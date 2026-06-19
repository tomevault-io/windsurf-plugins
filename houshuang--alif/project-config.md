---
trigger: always_on
description: A personal Arabic (MSA/fusha) learning app focused exclusively on reading and listening comprehension. No production/writing exercises. Tracks word knowledge at root, lemma, and conjugation levels using FSRS spaced repetition. Combines LLM sentence generation with deterministic rule-based validation (clitic stripping + known-form matching).
---

# Alif — Arabic Reading & Listening Trainer

## Project Overview
A personal Arabic (MSA/fusha) learning app focused exclusively on reading and listening comprehension. No production/writing exercises. Tracks word knowledge at root, lemma, and conjugation levels using FSRS spaced repetition. Combines LLM sentence generation with deterministic rule-based validation (clitic stripping + known-form matching).

**North-star metric:** genuinely-known words growing week over week (not activity/review volume). **The user's end goal is classical-literature breadth** — Quran, commentaries, medieval poetry, the full literary tradition — not just MSA. Curriculum and content choices should serve that, not generic frequency lists alone.

## Quick Start
```bash
# Backend
cd backend
cp .env.example .env  # add API keys
pip install -e ".[dev]"
python3 scripts/import_duolingo.py  # import 196 words
python3 -m uvicorn app.main:app --port 8000

# Frontend
cd frontend
npm install
npx expo start --web  # opens on localhost:8081
```

## Architecture
- **Backend**: Python 3.11+ / FastAPI / SQLite (single user, no auth, WAL mode, 30s busy_timeout) — `backend/`
- **Frontend**: Expo (React Native) with web + iOS mode — `frontend/`
- **SRS**: py-fsrs v6 (FSRS-6 with same-day review support) — `backend/app/services/fsrs_service.py`
- **TTS**: ElevenLabs REST, `eleven_multilingual_v2`, PVC voice. Audio cached by SHA256 in `backend/data/audio/`. Story audio in `backend/data/story-audio/`.
- **NLP**: Rule-based clitic stripping + known-form matching + CAMeL disambiguation + LLM disambiguation. See `docs/nlp-pipeline.md`.
- **Migrations**: Alembic for SQLite. Every schema change needs a migration. Auto-runs on startup.
- **Hosting**: Hetzner (46.225.75.29), venv + systemd (no Docker). Backend: systemd service `alif-backend`, port 3000, venv at `/opt/alif/backend/.venv/`. Frontend: systemd service `alif-expo`, port 8081. Spanish Pilot: systemd service `alif-spanish-pilot`, port 3100. Polyglot: systemd service `polyglot-backend`, port 3002, fronted by an in-app reverse proxy at `/polyglot/*` (`backend/app/routers/polyglot_proxy.py`, mounted in `backend/app/main.py`, forwarding to `127.0.0.1:3002`) so the client only needs alif's port 3000. DuckDNS: `alifstian.duckdns.org`. Data at `/opt/alif/backend/data/`. Limbic at `/opt/limbic` (PYTHONPATH), cost DB at `/opt/limbic-data/llm_costs.db`.
- **Spanish Pilot**: Standalone UX-validation prototype at `spanish-pilot/` — separate SQLite, separate systemd `alif-spanish-pilot` on port 3100 (`/opt/alif-pilot/`). Norwegian UI, no English. Tests Alif's word-level SRS + intro cards + memory hooks on 60 Norwegian school students learning Spanish. See `spanish-pilot/README.md`. Does NOT share any code with main Alif backend — completely isolated.
- **Polyglot**: Sister app at `polyglot/` for Modern Greek (primary), Ancient Greek, and Latin. Separate Python package (`polyglot-backend`), separate venv (`polyglot/.venv`), separate SQLite (`polyglot/polyglot.db`), separate FastAPI process on port 3002 (prod; 3001 dev). Reading-as-mapping primary UX (lazy PDF intake → tap unknowns → next-page presumes rest known). Uses simplemma for lemmatization with a dual-provider LLM-in-context quality gate — Codex `gpt-5.5` primary + Claude failover via `polyglot/app/services/llm_cli.py`. FSRS + acquisition Leitner + leech engine ported from Alif (`/api/reviews/{submit,introduce,due,stats}`); `mark_lemma(state='unknown')` enrols into Box 1 immediately. **Do NOT confuse `backend/` and `polyglot/`** — see `polyglot/CLAUDE.md` for project-specific rules. **Mirror Alif's design and code by default; divergence requires a specific Greek/Latin-driven reason recorded in the change. Alif is the product of 100+ days of real-user iteration — do not redesign it.** See `polyglot/CLAUDE.md` § "Ground design and code in Alif". Frontend (`frontend/`) talks to both; user picks via a Globe tab driven by `frontend/lib/language-context.tsx`. Plan: after ~6 weeks of dogfooding both languages, extract a shared `alif_core/` Python package (FSRS, acquisition, session builder) — premature now.
- **LLM Cost Tracking**: All litellm calls auto-logged via `limbic.cerebellum.cost_log` callback. Sync to local: `python -m limbic.cerebellum.cost_log sync`. Reports: `python -m limbic.cerebellum.cost_log report --days 7`.
- **Offline**: AsyncStorage sync queue for all mutable actions. Auto-prefetch, background refresh, 12s fetch timeout with stale-cache fallback. See `docs/frontend-files.md`.

## LLM Architecture

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [houshuang/alif](https://github.com/houshuang/alif) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
