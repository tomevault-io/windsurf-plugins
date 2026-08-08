---
trigger: always_on
description: Context for Codex. Read this before editing.
---

# CURB — SF street parking, block by block

Context for Codex. Read this before editing.

## What it is
A mobile-first web app that shows San Francisco street-parking rules on an
interactive map: where you can park, until when it's swept, whether it's
metered, and whether it's a residential permit (RPP) zone. Lets the user set a
calendar reminder before the next sweep.

## Stack (intentionally minimal)
- Single static file: `index.html`. No build step, no framework, no bundler.
- Vanilla JS + Leaflet 1.9.4 (from cdnjs) for the map.
- Basemap: official Google Map Tiles API when `GMAPS_KEY` (or `window.GMAPS_KEY`) is set —
  session-token flow in `initBasemap()`, viewport attribution refreshed on moveend. Falls
  back to keyless CARTO Voyager raster tiles when no key / on any failure. Leaflet stays the
  map engine either way. The Google key is a client key (referrer-restrict it) kept OUT of the
  public repo: local dev reads a gitignored `config.js` (from `config.example.js`); on Vercel,
  `api/config.js` emits `window.GMAPS_KEY` from the `GMAPS_KEY` env var and `vercel.json`
  rewrites `/config.js` → `/api/config`.
- Everything is client-side. Data is fetched live from DataSF (Socrata) at runtime.
- Design system: fonts Anton (display) + Hanken Grotesk (body); "transit signage"
  aesthetic; color tokens in :root (--green clear / --amber soon / --red now /
  --meter permit-blue / paper+ink). Keep this language if extending the UI.

## Data sources (all DataSF Socrata, CORS-open: `access-control-allow-origin: *`)
1. Street sweeping — `yhqp-riqs`
   https://data.sfgov.org/resource/yhqp-riqs.json
   Fields: cnn (segment id), corridor, limits (cross streets), blockside,
   cnnrightleft (L/R vs digitized direction), weekday, fromhour, tohour,
   week1..week5 ("1"/"0" = Nth occurrence of that weekday in the month),
   line (GeoJSON LineString). CURRENT data.
2. Parking meters — `8vzz-qzz9`
   https://data.sfgov.org/resource/8vzz-qzz9.json
   Fields: street_name (UPPERCASE), cap_color, on_offstreet_type, lat/long, etc.
   CURRENT data. Used only for a street-level count (no spatial join — see limits).
3. Parking regulations / RPP — `hi6h-neyh`
   https://data.sfgov.org/resource/hi6h-neyh.json
   Fields: regulation, rpparea1 (permit-area letter), hrlimit, days, from_time,
   to_time, exceptions, shape (GeoJSON MultiLineString). STALE: this is SFMTA's
   2017 set, flagged by the city as not comprehensively updated. Treat as a hint.

NOTE: enforcement + sweeper-pass data are now GPS-based — records request #26-5453 restored
GPS lat/long on citations (nearest-CNN-segment match, build-enforcement-records.py), and
records request #26-5451 supplies sweeper-pass times (data/sweeps.json, build-sweeps.py).

### Spatial queries (verified working)
- Segments in viewport: `?$where=intersects(line,'POLYGON((lng lat, ...))')&$limit=2500`
- RPP in viewport:      `?$where=intersects(shape,'POLYGON((...))') AND rpparea1 IS NOT NULL`
- Polygon ring order is `lng lat`, closed (first point repeated).
- Only fetched at map zoom >= 15 (MIN_ZOOM_DATA), debounced on `moveend`.

## Key product decisions / constraints (don't regress these)
- NO live space availability anywhere for SF — SFpark's sensor API was retired in
  2014. The app deliberately only shows *rules*, never "open spots." Don't add fake
  availability.
- The posted physical sign is the source of truth. Every detail sheet says so.
- Curb sides are drawn as two lines offset ~5 m (OFFSET) perpendicular to the
  centerline, signed by cnnrightleft (R=+1, L=-1; fallback alternate). offsetLine()
  uses a local equirectangular projection. Single-side blocks draw one centered line.
- "Next sweep" math = nextSweep(): iterates up to 70 days, matches weekday +
  Nth-occurrence-of-month flag, skips today's window if already past.
- Geolocation: navigator.geolocation is attempted but is often BLOCKED inside
  sandboxed preview iframes. Fallbacks: tap-the-map to drop "parked here", or search
  a street. Real GPS works once deployed / opened in a normal browser tab.

## File map
- index.html — the entire app (HTML + CSS + JS in one file).
- README.md — human-facing run/deploy notes.

## Run / deploy
- Local: just open index.html, or `npx serve .` for a localhost origin (better for
  geolocation testing).
- Deploy (static): `vercel` from this folder (zero config), or any static host.

## Roadmap — likely next task: push notifications
The calendar reminder (＋Reminder button → .ics with a 30-min VALARM) already covers
~90% of "remind me before sweeping" with zero backend. True push is the open item:
- Needs deployment + a service worker + Web Push (VAPID) subscription, and a tiny
  backend/cron (e.g. Vercel cron or Cloudflare Worker) to fire notifications at
  sweep-time minus N.
- iOS gotcha: Web Push only works when the site is installed to the Home Screen as a
  PWA (needs a manifest + service worker). Plan for an "Add to Home Screen" prompt.
- Persist the user's saved spot/schedule (localStorage is fine post-deploy; note it
  is intentionally NOT used in the in-chat artifact version).

## Other backlog ideas
- Pin meters per-block (requires spatial join of meters to sweeping segments;
  currently street-level count only).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alevizio/curb](https://github.com/alevizio/curb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
