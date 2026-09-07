---
trigger: always_on
description: Community-powered IPTV streaming application with **three clients** sharing a
---

# TV Viewer Project - Copilot Instructions

## Project Overview
Community-powered IPTV streaming application with **three clients** sharing a
common Supabase backend for channel data and analytics:

- **Desktop** (Python + CustomTkinter): Windows / Linux native app, VLC playback
- **Mobile** (Flutter 3.32): Android (Google Play), iOS scaffolding
- **Web / Docker** (FastAPI + vanilla JS): self-hosted server, browser UI

**Version:** 2.20.2 (semver, kept in sync via `config.py`, `flutter_app/pubspec.yaml`, and release workflows)
**License:** MIT
**Repo:** tv-viewer-app/tv_viewer (default branch: `master`)
**Distribution:** GitHub Releases (Windows zip, Linux x86_64, Android APK + AAB), Docker Hub (`asummoner/tvviewerapp:latest`), Google Play, F-Droid (MR open), APKPure (pending), Samsung (pending)
**Current release highlights:** Community Statistics page, timezone-based country detection, APKPure notify workflow
**Supabase project:** `cdtxpefohpwtusmqengu`

## Build, Test, and Lint Commands

### Desktop (Python)
```bash
pip install -r requirements.txt
python main.py                          # Run from source
python build.py                         # Build Windows EXE (PyInstaller)
python tests/validate_build.py --quick  # Pre-release validation
python -m pytest tests/ -q              # 299+ unit tests
```

### Web / Docker
```bash
python -m web.server                    # Run FastAPI dev server on :8765
docker build -t tv-viewer-web .         # Build container
docker compose up                       # Run with persistent /data volume
```

### Flutter (Android)
```bash
cd flutter_app
flutter pub get
flutter test                            # Dart unit tests
flutter build apk --release             # APK
flutter build appbundle --release       # AAB for Play Store
flutter analyze                         # Lint (release-blocking in CI)
```

## Tech Stack

| Layer | Technology | Notes |
|-------|------------|-------|
| Desktop UI | CustomTkinter + ttkbootstrap | Fluent 2 dark theme |
| Desktop player | VLC via python-vlc | No hardware-accel flags (instability) |
| Mobile | Flutter 3.32 + Material 3 | `video_player` for video, `just_audio` + `audio_service` for radio/background playback |
| Web backend | FastAPI + uvicorn | Single-file `web/server.py` |
| Web UI | Vanilla JS + HLS.js | `web/static/index.html` (~1500 LoC) |
| Shared DB | Supabase (Postgres + PostgREST) | Anon key client-side, RLS-protected |
| Analytics | Supabase `analytics_events` table | Opt-in only |
| HTTP client | aiohttp (async) + requests | certifi CA bundle in Docker |
| EPG | XMLTV (gzipped) from public sources | `utils/epg.py` |
| Build | PyInstaller (desktop) / Flutter 3.32.0 / Docker buildx | Android uses Gradle 8.9, AGP 8.7.0, Kotlin 1.9.22 |
| CI/CD | 28 GitHub Actions workflows | Release gate, store distribution, Docker publish, analytics and Supabase ops |

## Architecture

### Desktop (Python)
- `core/channel_manager.py` — single source of truth for in-memory channel list;
  `__slots__` for memory; RLock-protected; URL→index map for O(1) lookups
- `core/repository.py` — async M3U fetcher (aiohttp), configurable sources
- `core/stream_checker.py` — background asyncio validation in daemon thread
- `ui/main_window.py` — Tk main loop; **all UI updates from background threads
  MUST use `root.after(0, ...)`** or segfault
- `ui/player_window.py` — VLC player; Space/F/M/Esc keyboard shortcuts

### Flutter (Mobile)
- `flutter_app/lib/services/` — `analytics_service.dart`, `shared_db_service.dart`,
  `audio_handler.dart` (background playback), `epg_service.dart`
- `flutter_app/lib/screens/` — `home_screen.dart`, `player_screen.dart`,
  `radio_screen.dart`, `statistics_screen.dart`, `help_support_screen.dart`
- Supabase credentials injected via `--dart-define=SUPABASE_URL=…` at build time
- Pinned HTTPS via `lib/utils/pinned_http_client.dart` for *.supabase.co

### Web / Docker
- `web/server.py` — FastAPI app. Endpoints: `/api/channels`, `/api/refresh`,
  `/api/health/report`, `/api/sources/{name}`, `/api/epg/{channel}`, `/api/statistics`, `/proxy`
  (SSRF-protected — blocks private IP ranges)
- `web/static/index.html` — SPA, localStorage state, HLS.js, EPG overlay
- Persistent state in `DATA_DIR` (`/data` volume in Docker): channels.json,
  favorites.json, epg_cache.json
- `utils/supabase_channels.py` — shared DB client: fetch, report broken,
  report working, and serve community statistics
- `utils/normalize.py` — **single source of truth** for category/country
  normalization (14 canonical categories). Ported by hand to Dart; risks drift.

### Shared Backend (Supabase)
- **Project ref:** `cdtxpefohpwtusmqengu`
- `channels` — `url_hash`, `name`, `urls[]`, `category`, `country`, `logo`,
  `media_type`, `source`, etc.
- `channel_status` — `url_hash`, `status` (working/broken), `last_checked`,
  `report_count`
- `analytics_events` — opt-in usage telemetry (sessions, plays, failures)
- `v_channels_with_sources` — view with `security_invoker=true` (RLS honors caller)
- `mv_daily_active_users` — materialized view for aggregated usage data
- `/api/statistics` reads pre-aggregated anonymous data only; clients never query analytics tables directly

## Critical Conventions

### Threading (Desktop)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tv-viewer-app/tv_viewer](https://github.com/tv-viewer-app/tv_viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
