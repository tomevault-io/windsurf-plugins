---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Single-process service that connects to a BEAST TCP feed (readsb /
dump1090-fa, typically port 30005), decodes Mode S / ADS-B messages,
and exposes: a Leaflet map at `/`, a floating detail panel for
per-aircraft info, HTTP API, `/ws` WebSocket pushing 1 Hz snapshots,
and (optional) per-message JSONL logging to `/data/beast.jsonl`.
Snapshots are enriched from several free public sources:

- Routes (callsign → airports) and per-tail details (registration, type,
  operator, photo URL) from [adsbdb.com](https://www.adsbdb.com/).
- Aircraft photos from [planespotters.net](https://www.planespotters.net/)
  (with adsbdb's airport-data.com URL as fallback).
- Airline IATA code + alliance from
  [OpenFlights](https://openflights.org/data.html)'s `airlines.dat`.
- Airport names + coordinates from [OurAirports](https://ourairports.com/).
- METAR weather at origin / destination from
  [aviationweather.gov](https://aviationweather.gov/).

A server-side watchlist + notification fan-out (Telegram / ntfy / generic
webhook) fires alerts on watched-tail reappearance and emergency squawks
(7500 / 7600 / 7700), whether a browser tab is open or not. Channels are
UI-managed, persisted to `/data/notifications.json`.

## Repository layout

```
dotnet/                 # Backend solution
  src/
    FlightJar.Api/              # Minimal API, DI, BackgroundServices, HTTP + WS endpoints
    FlightJar.Core/             # Registry, state, enrichment, phase/route logic,
                                # reference-data loaders, stats (polar coverage, traffic heatmap)
    FlightJar.Decoder/          # BEAST framing + Mode S + CPR (no framework deps)
    FlightJar.Clients/          # Typed HTTP clients: adsbdb, planespotters, metar, vfrmap, openaip
    FlightJar.Notifications/    # INotifier + Telegram/Ntfy/Webhook + dispatcher + AlertWatcher
    FlightJar.Persistence/      # Gzipped JSON state, watchlist, notifications config
    FlightJar.Terrain/          # SRTM tile store + line-of-sight solver (no framework deps)
  tests/
    FlightJar.Api.Tests/        # WebApplicationFactory integration + BEAST-replay E2E
    FlightJar.Core.Tests/
    FlightJar.Decoder.Tests/    # BEAST parser + ModeS/CPR vs pyModeS golden vectors
    FlightJar.Clients.Tests/
    FlightJar.Notifications.Tests/
    FlightJar.Persistence.Tests/
    FlightJar.Terrain.Tests/
  FlightJar.slnx              # Solution (slnx format)
  Directory.Build.props       # Shared compiler settings
  global.json                 # SDK pin
  Dockerfile                  # Multi-stage build, produces the flightjar image
app/
  static/                     # Leaflet frontend — ES modules, served verbatim
scripts/
  fetch_plane_shapes.py       # Build-time utility: generates app/static/tar1090_shapes.js
tests/
  js/                         # Frontend unit tests (`node --test`)
  e2e/                        # Playwright smoke suite against the running backend
docker-compose.yml            # Builds via dotnet/Dockerfile, context = .
.github/workflows/ci.yml      # check + e2e + publish jobs
```

## Common commands

```bash
# Build + run the container (reads docker-compose.yml for BEAST_HOST etc.)
docker compose up --build -d
docker compose logs -f flightjar
docker compose down

# Run the app directly (without Docker)
BEAST_HOST=localhost BEAST_PORT=30005 LAT_REF=52.98 LON_REF=-1.20 \
  dotnet run --project dotnet/src/FlightJar.Api --urls http://127.0.0.1:8080

# Dev loop
cd dotnet
dotnet format FlightJar.slnx --verify-no-changes
dotnet build FlightJar.slnx
dotnet test FlightJar.slnx
cd ..
node --test tests/js/         # frontend ES-module tests (Node 20+)
npx playwright test           # Playwright smoke against live backend
```

## Architecture

Single entry point: `dotnet/src/FlightJar.Api/Program.cs`. Everything —
BEAST consumer, registry worker, snapshot pusher, WebSocket broadcaster,
external clients, watchlist / notifications, alert watcher — runs inside
one process under the Generic Host. Shared state is owned by a single
writer thread (`RegistryWorker`) so the `AircraftRegistry` doesn't need
locks.

### Data flow

1. **`BeastConsumerService : BackgroundService`** opens a TCP connection
   to `BEAST_HOST:BEAST_PORT`, wraps it in a `PipeReader`, and parses
   frames with `BeastFrameReader.ParseMany` (`ParseOne` uses
   `stackalloc Span<byte>` for zero-alloc escape unescaping). Frames
   are written to a bounded `Channel<BeastFrame>` with drop-oldest
   overflow. Auto-reconnect with exponential backoff (1s → 30s).
2. **`RegistryWorker : BackgroundService`** owns the registry. Three
   concurrent tasks (frame pump, 1 Hz ticker, work-loop consumer)
   serialise through one `Channel<Work>` so all mutations happen on a
   single thread. Each tick: cleanup stale aircraft → build base
   snapshot via `registry.Snapshot(now)` → enrich (origin/destination
   from adsbdb cache, airport info, flight phase, airline + alliance,
   route plausibility gate) → publish to `CurrentSnapshot` (volatile
   ref for HTTP readers) → fan out via `SnapshotBroadcaster`.
   Additional per-tick hooks: watchlist last-seen bookkeeping, alert

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MrSuttonmann/flightjar](https://github.com/MrSuttonmann/flightjar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
