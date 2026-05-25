---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## What This Is

A Python 3 pipeline that exports a Plex library, feeds it to an LLM for channel curation, and creates themed virtual TV channels in [Tunarr](https://github.com/chrisbenincasa/tunarr).

Available as a **Docker web app** (primary) or a **CLI tool** (power users / advanced).

## Recommended Entry Point — Web UI (Docker)

```bash
docker compose up -d
# then open http://<host-ip>:7979
```

First run shows an onboarding wizard: create login credentials → enter Tunarr/Plex/TMDB URLs → dashboard. Config is saved to `./data/config.json` (bind-mounted volume).

## CLI Entry Point (power users)

```powershell
python programmarr.py
```

`programmarr.py` is the interactive CLI wrapper. It handles first-time config setup,
walks through the full workflow for whichever path the user picks, always runs a probe
before deploying, and offers Plex sync at the end.

## Web UI Architecture

**Stack:** FastAPI (Python) + React + Mantine v7 — served as a single Docker container on port 7979.

**Directory layout:**
```
backend/          FastAPI app + routers
  main.py         Entry point — auth middleware, SPA fallback, lifespan
  routers/
    config_router.py    GET/POST /api/config, /api/config/status
    status_router.py    GET /api/status (Plex+Tunarr ping), /api/tunarr/channels
    channels_router.py  CRUD /api/channels, /api/channels/{n}, /api/library/titles
    pipeline_router.py  SSE-streaming pipeline endpoints (export, probe, deploy, deploy-selective, collections, etc.)
    logs_router.py      GET /api/logs, /api/logs/{name}
frontend/         React + Mantine SPA (built to backend/static/)
  src/pages/
    Onboarding.tsx  First-run wizard (shown when config.json missing/unconfigured)
    Dashboard.tsx   Live Tunarr channel grid + connection status
    Run.tsx         Pipeline stepper — AI / No-AI / Collections tabs
    Channels.tsx    channels.json editor (Tier 2: click-to-edit)
    Settings.tsx    config.json editor (masked sensitive fields)
    Logs.tsx        Per-run log viewer
data/             Bind-mounted volume — config.json, channels.json, plex_library.csv, logs/
```

**Environment variables (Docker):**
- `PROGRAMMARR_DATA` — path where data files live (default: `/data`)
- `PROGRAMMARR_SCRIPTS` — path where Python scripts live (default: `/app`)

**Key design decisions:**
- Pipeline scripts (`export.py`, `create.py`, etc.) run as subprocesses with `cwd=DATA_DIR` so their relative file opens work correctly without modification
- SSE (Server-Sent Events) streams subprocess stdout line-by-line to the browser inline terminal
- Auth middleware reads `config.json` on every request — no restart needed to enable/disable auth
- Onboarding shown automatically when `config_status.configured` is false (no Tunarr/Plex/token set)
- Channels page reads from `channels.json` (local file), Dashboard reads live from Tunarr API
- `asyncio.WindowsProactorEventLoopPolicy` is set at startup in `main.py` — required on Windows for `asyncio.create_subprocess_exec` to work; no-op on Linux/Docker
- **Deferred (Tier 3):** drag-to-reorder channels, autocomplete from plex_library.csv, inline Plex validation

## Local Development (Docker)

The recommended local dev loop is Docker — it gives exact production parity and avoids Windows asyncio/subprocess issues:

```powershell
# From repo root — builds frontend, bakes into image, runs on localhost:7979
docker compose build && docker compose up
```

The `docker-compose.yml` mounts `./data` as a volume, so your `config.json`, `channels.json`, and `plex_library.csv` persist between runs. To pick up code changes, rebuild: `docker compose build && docker compose up`.

Two environments:
- **localhost:7979** — local Docker build for testing before pushing
- **TrueNAS** — production, pulls image from GHCR automatically on every `master` push via GitHub Actions

## Workflow

```
export.py  ->  LLM (Gemini/Claude/ChatGPT)  ->  create.py
               or
export.py  ->  generate_no_ai.py  ->  create.py
```

Plex collections (managed by Kometa/Trakt/Letterboxd) can be turned into
channels directly without the export/LLM step:

```
generate_from_collections.py --apply  ->  create.py
```

## Running the Scripts (advanced / direct use)

```powershell
# Step 1 — export Plex library to CSV
python export.py

# Step 2a — AI path: paste plex_library.csv + PROMPT.md into any LLM, save output as channels.json
# Step 2b — no-AI path: auto-generate starter channels.json from metadata
python generate_no_ai.py

# Step 2c — collection path: generate one channel per Plex collection (80+ block)
python generate_from_collections.py              # preview
python generate_from_collections.py --apply      # write to channels.json
python generate_from_collections.py --condense   # skip collections matching existing channel names
python generate_from_collections.py --min-items 5  # skip tiny collections
python generate_from_collections.py --base 90    # start at channel 90 instead of 80

# Step 3 — create channels in Tunarr
python create.py --probe    # dry run first
python create.py            # apply
```

## Configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlpineArchitecture/programmarr](https://github.com/AlpineArchitecture/programmarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
