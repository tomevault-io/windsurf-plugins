---
trigger: always_on
description: Guidance for working in this repo. See [README.md](README.md) for the full
---

# CLAUDE.md

Guidance for working in this repo. See [README.md](README.md) for the full
hardware/setup story; this file covers what's non-obvious when editing the code.

## What this is

A headless Raspberry Pi 3B drives a 128×64 HUB75E RGB LED matrix (Adafruit RGB
Matrix Bonnet #3211) to show next VBZ bus departures from
`transport.opendata.ch`, grouped by station, with a clock, optionally the
inside/outside temperature from an Ecowitt weather gateway (GW3000) on the LAN,
and horizontally scrolling destination labels.

## Architecture (data flow)

`config.json` → `config.py` (load/validate) → poller thread in `__main__.py`
fetches each station once/minute via `transport.py` (plus the weather gateway
via `weather.py`, when configured) → shared state under one lock holds raw
`Departure`s + the latest `Temps` → render loop (30fps while a label scrolls,
4fps idle) builds `StationGroup`s/`TempReadout` (`layout.py` composes a PIL
frame) → `renderer.py` pushes it to the panel.

| File | Responsibility |
|------|----------------|
| `src/config.py` | Parse/validate `config.json`; dataclasses `Config/Station/Connection/Display/Weather`. Resolves each connection's on-screen `label`. |
| `src/transport.py` | API client + parsing. `parse_departures` matches on `(number, to)`; `visible_departures` applies `min_time` + live countdown. No rendering. |
| `src/weather.py` | Ecowitt gateway client: `fetch_weather` GETs `get_livedata_info`, `parse_livedata` extracts outdoor (`common_list` id `0x02`) + indoor (`wh25[0].intemp`) into `Temps`. No rendering. |
| `src/layout.py` | **Pure PIL, no `rgbmatrix` import.** `FrameComposer.compose()` builds the 128×64 image; holds fonts, column geometry, and scroll state. |
| `src/renderer.py` | **Only** module importing `rgbmatrix`. Thin wrapper: options + `SetImage`/`SwapOnVSync`. |
| `src/__main__.py` | `Poller` thread + render loop + signal-based shutdown. Entry: `python3 -m src`. |
| `server/` | FastAPI backend for the config web UI. Reuses `parse_config` (validation) + `FrameComposer` (PNG preview); **never** imports `renderer`/`rgbmatrix`. Entry: `python3 -m server` (own venv `webenv/`, own service on :8080). |
| `web/` | React+TS config UI (Vite). Built on the dev machine; `web/dist/` is committed and served by `server/`. |

## Key conventions / gotchas

- **rgbmatrix lives in a venv, not system Python.** Adafruit's installer refuses
  Debian's externally-managed Python, so `setup_pi.sh` builds the bindings into a
  `--system-site-packages` venv at `env/` and rewrites the systemd unit's
  `ExecStart` to `env/bin/python3 -m src`. The venv still sees apt-installed
  Pillow/requests. Don't switch the service back to `/usr/bin/python3`.
- **`rgbmatrix` only on the Pi.** Keep its import isolated to `renderer.py`. Tests,
  `layout.py`, and `tools/preview.py` must stay importable on the dev machine
  (which has no `rgbmatrix`). If you need new render logic testable off-Pi, put it
  in `layout.py`, not `renderer.py`.
- **Direction matching:** `connections[].destination` is matched against the API
  `to` field (the line's terminal *in the desired direction*), not an intermediate
  stop. E.g. line 32 → `"Zürich, Strassenverkehrsamt"` selects that direction.
- **Countdown is live between polls.** The API is polled once/minute, but minutes
  and the `min_time` filter are recomputed every frame from cached
  `departure_ts` (the API's realtime estimate when present, else planned). Don't
  move that logic into the poller. A station whose last successful poll is older
  than `STALE_POLLS` intervals renders dimmed (`StationGroup.stale`), so an
  outage doesn't masquerade as a live board.
- **Weather is optional and config-gated.** An empty/absent `weather.url`
  disables it entirely (the poller never calls the gateway). Temps are drawn in
  the header font left of the clock, indoor (`temp_in` colour) then outdoor
  (`temp_out`), one decimal; a `None` value (unpaired sensor) is skipped, and the
  first station header clips short of the block. Staleness reuses the station
  rule (`_stale_after`) → dimmed `TempReadout`; nothing is drawn before the first
  successful fetch.
- **Fonts are BDF → PIL.** Pillow can't read `.bdf` at draw time, so `layout.py`
  compiles them once (`BdfFontFile`) into a temp cache. PIL *bitmap* fonts have no
  `getmetrics()`; use `font_height()`/`text_w()` (getbbox/getlength) instead.
- **Scrolling:** per-row x-offset keyed by `station_id|number|label`, advanced by a
  wall-clock delta clamped to `MAX_DT` (avoids jumps after a frame stall). The
  destination column is pixel-clipped via a sub-image paste, not `fit_text`
  truncation (that's only for headers).
- **The live `config.json` is untracked.** The repo commits `config.example.json`;
  `setup_pi.sh` copies it to `config.json` on first install and the web UI
  rewrites the live file at runtime (`PUT /api/config`, validated via
  `parse_config`, keeps a `.bak`) — which is exactly why it can't be tracked:
  web-UI saves must not dirty the Pi's checkout and break `git pull` deploys.
  Server `GET /api/config`, `tools/preview.py`, and the tests all fall back to
  the example when the live file is missing. Don't commit a live config.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pierrecattin/transport_display](https://github.com/pierrecattin/transport_display) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
