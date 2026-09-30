---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

> **Keep this lean and current.** Point at code and docs; don't restate them — paraphrased
> code is the #1 source of drift. Behavior changes update this file in the **same commit**.
> History → `CHANGELOG.md`; full designs → `docs/`; exhaustive endpoint/flag reference →
> [`docs/api.md`](docs/api.md) & each script's `--help`. If a section outgrows its job,
> relocate the detail and leave a pointer.

## What This Is

A Python 3 pipeline + web app that exports a Plex library, **composes curated themed
virtual TV channels** in a deterministic Planner (with an optional AI layer on top),
and deploys them to [Tunarr](https://github.com/chrisbenincasa/tunarr). Channels can
be marked **live** to auto-update as the library grows.

The web app's channel-creation experience is a single **Planner** (`Run.tsx`): pick
genres/decades "in play," then check exact curated candidates — per-show marathons,
genre×decade cuts, named sub-genres, studio/director/actor channels, **TV network channels**
(from the `Studio` CSV column for TV rows), **classic programming blocks** (matched from
`programming_blocks.json`), and **franchise channels** (detected from TMDB
`belongs_to_collection` + Wikidata series/franchise membership, on-demand + cached, with per-member checkboxes) — built
deterministically via `/pipeline/compose`. An optional "✨ Bring in AI" layer adds
*discovery* (themed channels filters miss) and *tonal curation* (split a broad pool by
vibe), merged on top.

Two entry points: a **Docker web app** (primary — FastAPI + React on port 7979) and
an interactive **CLI** (`python programmarr.py`, for power users — first-run config
setup, always probes before deploying, offers Plex sync at the end).

> **Audience:** user-facing docs (install, quick start, screenshots) live in
> [`README.md`](README.md). **This file is the developer/agent reference** —
> architecture, conventions, the rules an agent must not break. It describes **what
> exists today**; planned/unbuilt ideas go in [`docs/ideas.md`](docs/ideas.md).

## Web UI Architecture

**Stack:** FastAPI (Python) + React + Mantine v7 — served as a single Docker container on port 7979.

**Directory layout:**
```
backend/          FastAPI app + routers
  main.py         Entry point — auth middleware, SPA fallback, lifespan, scheduler start
  scheduler.py    In-process asyncio loop for live channels (see Live Channels)
  routers/        config / status / channels / pipeline / recipes / logs routers
frontend/         React + Mantine SPA (built to backend/static/)
  src/pages/      Onboarding, Dashboard, Run (the Planner stepper), Channels, Settings, Logs
data/             Bind-mounted volume — config.json, channels.json, plex_library.csv, logs/
```

**Environment variables (Docker):**
- `PROGRAMMARR_DATA` — path where data files live (default: `/data`)
- `PROGRAMMARR_SCRIPTS` — path where Python scripts live (default: `/app`)

**Key design decisions (non-obvious — don't undo these):**
- Pipeline scripts (`export.py`, `create.py`, etc.) run as subprocesses with `cwd=DATA_DIR` so their relative file opens work unmodified.
- SSE (Server-Sent Events) streams subprocess stdout line-by-line to the browser inline terminal. `_stream` **must always end in a `done` event** — the UI spins until it sees one, so a failed subprocess launch, a dead pipe, or an unwritable log each yield `done` (with `returncode: -1` for the first two) rather than killing the generator mid-response.
- Auth middleware reads `config.json` on every request — no restart needed to enable/disable auth.
- Onboarding shows automatically when `config_status.configured` is false (no Tunarr/Plex/token set). It offers **Test connections** (`POST /api/test-connection`), which validates the typed-but-unsaved values — saving alone never contacts Tunarr or Plex, so without this a typo'd URL produced a green "Setup complete". A failed test warns but still lets the user continue ("Continue anyway") — never trap someone behind a probe.
- **Dashboard** shows an EPG guide grid (fetched via `GET /api/guide` → Tunarr XMLTV). Clicking a channel navigates to its editor.
- **Channels page** lists channels from the live Tunarr API (`GET /api/tunarr/channels`); clicking a row fetches the full `channels.json` entry and opens the editor. Channels in Tunarr with no `channels.json` entry show as **"Not managed by Programmarr"** (read-only orphans).
- **Save and Apply** (`POST /api/channels/{number}/apply`) saves a channel edit to `channels.json` and pushes it to Tunarr in place — preserving the Tunarr id and Plex DVR mapping. This is the Channels-page equivalent of the scheduler's per-channel update, but available for any channel (not just live ones).
- `asyncio.WindowsProactorEventLoopPolicy` is set at startup in `main.py` — **required** on Windows for `asyncio.create_subprocess_exec`; no-op on Linux/Docker. (This is the one place it's stated; don't duplicate it.)
- **Deferred (Tier 3):** drag-to-reorder channels, autocomplete from plex_library.csv, inline Plex validation.

## Local Development


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamesmattson/programmarr](https://github.com/jamesmattson/programmarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
