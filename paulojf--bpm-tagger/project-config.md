---
trigger: always_on
description: Before taking any action that could incur charges on the team plan — including but not limited to: making API calls, spawning paid agents (e.g. /ultrareview), running cloud services, using billable MCP tools, or any other metered resource — always warn the user and ask for explicit confirmation before proceeding.
---

# BPM Tagger — Master Project Context

## Cost Guardrail (ALWAYS ENFORCE)

Before taking any action that could incur charges on the team plan — including but not limited to: making API calls, spawning paid agents (e.g. /ultrareview), running cloud services, using billable MCP tools, or any other metered resource — always warn the user and ask for explicit confirmation before proceeding.

---

## Project Identity

| Field | Value |
|---|---|
| Name | BPM Tagger |
| Version | v2.3.1 |
| GitHub | https://github.com/PauloJf/BPM-Tagger (public since 2026-05-27) |
| Docker Hub | `gatoserio/bpm-tagger:latest` (slim) · `gatoserio/bpm-tagger:full` (PyTorch/deeprhythm) |
| Author | Paulo (paulo@gatoserio.dev) |
| Git identity | Always use `paulo@gatoserio.dev` / `Paulo` for all commits on this project |
| Origin | Built by a runner who needed BPM-tagged music for cadence-synced running |

---

## What It Does

Automatically detects the BPM of every track in a [Navidrome](https://www.navidrome.org/) music library, writes the result back to the file's metadata tag, tracks everything in a SQLite database, sends batched [ntfy](https://ntfy.sh/) notifications, and exposes a password-protected web UI for reviewing and correcting results.

---

## Repository Layout

| File / Dir | Purpose |
|---|---|
| `bpm_tagger/` | Package: entry point (`python -m bpm_tagger`), config, db, `bpm/`, `scan/`, `notify/`, `integrations/`, `web/` |
| `web_ui.py` | Back-compat shim → re-exports `bpm_tagger.web.app.start` / `create_app` |
| `requirements.txt` | Python dependencies |
| `Dockerfile` | Multi-stage build; `WITH_DEEPRHYTHM=true` for full image |
| `docker-compose.yml` | Primary deployment path |
| `.github/workflows/docker-publish.yml` | Manual CI — builds & pushes both image variants |
| `VERSION` | Single source of version truth for CI |
| `README.md` | Full user-facing documentation |
| `DOCKERHUB_README.md` | Docker Hub short description |
| `CHANGELOG.md` | Version history |
| `docs/screenshots/` | UI screenshots used in README |

---

## Architecture

### Entry Point & Thread Model

```
bpm_tagger.main()
  ├── loads config from env + /data/settings.json
  ├── creates BPMTagger (owns BPMDatabase + NotificationManager)
  ├── [daemon thread] web_ui.start()  ← Flask/Waitress on :5000
  └── runs scan / watch / report / lock / unlock based on MODE
```

### BPM Detection Pipeline

Three detectors run in order per file:

1. **deeprhythm** — PyTorch CNN; lazy-loaded per worker thread. Full image only.
2. **essentia** — `RhythmExtractor2013(method="multifeature")` at 44 100 Hz; optional, fails gracefully.
3. **librosa** — always present; multi-segment analysis, returns median BPM + beat-consistency confidence.

Results → `_reconcile()` → `(final_bpm, needs_review)` → `_normalize_bpm()` into `[BPM_MIN, BPM_MAX]`.

### Scan Phases

1. **Discovery** — bulk-registers all audio files as `status='pending'` (skips locked/unchanged). Library is immediately visible in UI.
2. **Processing** — `ThreadPoolExecutor` processes all `pending` rows. `_pause_event` / `_stop_event` control pause/stop.

`file_hash` = `size:mtime` (not content hash) for speed. Hash is re-read after tag write to avoid re-analysis loop.

### Database (`BPMDatabase`)

SQLite, WAL mode. Schema migrations are additive `ALTER TABLE ADD COLUMN` in `_migrate()` — safe on existing DBs.

Key columns: `file_path`, `file_hash`, `bpm` / `bpm_dr` / `bpm_es` / `bpm_lb`, `bpm_confidence`, `detector`, `status` (`pending` / `done` / `error`), `needs_review`, `reviewed`, `locked`, `waveform_peaks` (JSON, lazy-computed).

### Web UI (`web_ui.py`)

- Flask served by Waitress on `UI_PORT` (default 5000)
- All state-changing routes require CSRF token (`_check_csrf()`)
- File paths validated against `MUSIC_DIR` (`_assert_in_music_dir()`)
- Runtime settings saved to `/data/settings.json` via `_save_settings()` — no restart needed for most changes
- Restart via `/api/restart` uses `os.execv()` (in-place process replacement)
- Waveform cache: in-memory dict → DB `waveform_peaks` → on-demand librosa recompute (deduplicated via `threading.Event`)

### Settings Persistence

`/data/settings.json` overrides env vars at startup. Env vars = defaults; UI = source of truth after first write.

### Notifications (`NotificationManager`)

Batches ntfy pushes: fires on `NTFY_BATCH_SIZE` tracks accumulated **or** `NTFY_MIN_INTERVAL` seconds elapsed. Watch mode flushes every 60 s. Scan summaries and review reports are separate messages.

---

## Docker Image Variants

| Build arg | Tag | Detectors | Peak RAM |
|---|---|---|---|
| `WITH_DEEPRHYTHM=false` (default) | `:latest` | essentia + librosa | ~400 MB |
| `WITH_DEEPRHYTHM=true` | `:full` | deeprhythm + essentia + librosa | ~1.8 GB |

`USE_DEEPRHYTHM` env var controls runtime usage (detector must also be present in the image).

---

## Key Commands

### Docker (primary)

```bash
# Build and start (slim)
docker compose up -d --build

# Build full image

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PauloJf/BPM-Tagger](https://github.com/PauloJf/BPM-Tagger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
