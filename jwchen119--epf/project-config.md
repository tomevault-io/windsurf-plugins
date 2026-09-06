---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An e-paper photo frame split across two codebases that talk over HTTP:

- **Server** (`app.py`, `epf/`, `cpy.pyx`, `templates/`, `static/`) — a Flask app, normally run in Docker on a NAS. Pulls photos from an [Immich](https://immich.app) album, crops/enhances/dithers them to the panel's 6-color palette, and serves the result already packed for the display.
- **Firmware** (`Arduino/`) — an ESP32-C6 sketch (`epd7in3e.ino`) for a Waveshare 7.3" Spectra 6 (E6) panel, 800x480. It does no image processing: it streams bytes straight into the panel and goes back into deep sleep.
- **CAD** (`CAD/*.STEP`) — enclosure parts, not built by any toolchain here.

There are no tests, no linter config, and no build system for the Python side.

## Server: build and run

`docker compose up -d`, with a `.env` alongside holding `IMMICH_API_KEY`. Or directly: `python app.py` (serves on `0.0.0.0:5000`).

`docker-compose.yml` exists because two things are easy to get wrong and both fail silently:

- Two paths are **hardcoded**, not configurable: `/config/config.yaml` (written by the settings page, watched by `watchdog` for external edits, created from `DEFAULT_CONFIG` if missing) and `/photos` (override with `IMMICH_PHOTO_DEST`; holds only `tracking.txt` — no photos are ever written to disk). A plain `docker restart` keeps them, but recreating the container — which is what updating the image requires — discards anything not bind-mounted, so settings revert to `DEFAULT_CONFIG` without any error.
- **`TZ` must be set.** The base image has no timezone, so `datetime.now()` returns UTC. `/sleep` derives both the sleep window and the wake-up schedule from local time, so an unset `TZ` shifts the frame's quiet hours by the whole UTC offset. Zone data is already in the image, so the env var alone is enough — no `tzdata` install and no `/etc/localtime` mount.

`IMMICH_API_KEY` is read once at import into the module-level `headers` dict. Note the README's `docker run` example writes `IMMICH-API-KEY` with hyphens, which the app does not read.

## Server: how the code is laid out

`app.py` holds the Flask app and every route; everything else is in `epf/`:

| module | holds |
| --- | --- |
| `config.py` | `DEFAULT_CONFIG`, the live settings, `config.yaml` read/write, the watchdog observer |
| `state.py` | in-memory runtime state: battery reading, current photo, pre-chosen next photo |
| `eventlog.py` | the JSONL event log and `client_ip()` |
| `tracking.py` | `tracking.txt` |
| `immich.py` | album/asset queries, `select_asset()`, health check, thumbnail fetch |
| `imaging.py` | the pipeline; takes its settings as arguments and reads no globals |
| `battery.py` | voltage → percentage |
| `notify.py` | low-battery push over Telegram or LINE |
| `credentials.py` | the notification tokens, in `/config/credentials.json` |

**The live settings are one dict that is only ever updated in place** (`config.apply()` calls `.update()`; it never rebinds `config.current`). Modules call `config.immich()` at the point of use. Copying a value out at import time — `from epf.config import current; url = current['immich']['url']` — would freeze it at start-up, which is the one way to break this layout. `config.current` is a `deepcopy` of `DEFAULT_CONFIG` for the same reason a shallow copy was wrong: the two would share the inner dict, and saving settings would rewrite the defaults the reset button restores.

The front end is split the same way: `templates/settings.html` is markup only, with `static/css/settings.css`, `static/js/i18n.js` (the translation dictionary) and `static/js/settings.js` (behaviour). Two things stay inline in the template and must: the theme bootstrap has to run before the stylesheet or the wrong theme flashes, and `window.EPF_DEFAULTS` is how the server's defaults reach the static JS. Assets are linked through `static_url()`, which appends the file's mtime so a browser cannot serve a stale one after an update.

## Server: HTTP contract with the firmware

This contract is the thing to be careful about — both sides must change together.

- `GET /download` — the device sends its battery voltage in a `batteryCap` **request header** (millivolts). Response is `text/plain`: ASCII hex bytes as `"XX,XX,..."` terminated by `};`, i.e. C-array source text, not binary. The `X-Photo-Url` response header carries the Immich web URL of the chosen photo (intended for writing an NFC tag; the firmware does not read it yet).
- `GET /sleep` — returns `{current_time, next_wakeup, sleep_duration}` where `sleep_duration` is **milliseconds**. The firmware divides by 1000 and passes it to `esp_deep_sleep`. Falls back to 24h if absent. `/download` and `/sleep` are two separate requests per wake cycle.
- `GET /setting` (GET renders, POST saves) — the config UI; `/` redirects here. Battery percentage shown here comes from the last `/download` request's header, cached in module globals for one hour, so it reads 0% until the device has checked in.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jwchen119/EPF](https://github.com/jwchen119/EPF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
