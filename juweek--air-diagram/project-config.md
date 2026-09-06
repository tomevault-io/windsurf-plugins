---
trigger: always_on
description: A single-purpose data-tool page. You search a ZIP or city (or pick a scenario)
---

# What's in the Air — agent handoff

A single-purpose data-tool page. You search a ZIP or city (or pick a scenario)
and it draws that air **particle by particle** with a live p5 canvas, next to a
readout that makes one editorial point:

> The green "Good" badge hides two things — a **legal line that's far looser
> than the health line**, and **what's actually in a breath**.

It's built on the Gourmet Data project template (see `README.md` for the generic
template docs). This file is the project-specific map: how it's wired, and the
priorities that should break ties when you change it.

---

## Priorities (read these first — they decide design questions)

1. **Simplicity & maintainability.** Idiomatic React/JS, plain functions over
   clever abstractions. This is a small site; keep it small. A new data source,
   viz, or scenario should be *one file following a convention*, not an edit to a
   central engine.
2. **Honesty about the data.** This is the whole soul of the piece. The official
   PM2.5 number is a single mass figure — it does **not** say what the particles
   are or where they came from, and ultrafine particles aren't in it at all. So:
   - The source split and the ultrafine swarm are **modeled, not measured**, and
     the UI must always say so (`site.config.js` attribution note, the legend
     footnotes). Never let a modeled number read as a measurement.
   - **Zero must mean zero.** If an input pollutant is absent, its source must
     read ~0 — no artificial baselines conjuring particles. (We already fixed
     one over-inclusion bug here; don't reintroduce additive floors in
     `composition.js`.)
   - Scenarios (cigarette, wildfire, …) use **illustrative published
     concentrations**, clearly labeled as such, not live data.
3. **Clarity of copy.** The reader should never be confused about what a number
   means. Percentages sum to 100 over the regulated sources; ultrafine is shown
   as an extra count *outside* that 100 with an explanation; pollutants are
   framed against the bulk gases (a breath is ~99.9% N₂/O₂/Ar). When in doubt,
   explain more.
4. **The narrative comes through the visuals.** The gap between the **US legal
   line** and the **WHO health line** should be visible, not just stated — e.g.
   the by-pollutant view shows both lines at once; the two ring canvases (WHO
   dense vs legal sparse) show the same air judged two ways.
5. **Polish & containment.** Nothing escapes its parent bounds; works at mobile
   width; active controls are unmistakable (solid black); the canvas stays
   performant. The owner iterates **visually** — verify changes in the browser
   and share screenshots, don't just assert they work.
6. **Every state is a URL.** `/Detroit`, `/90001`, `/cigarette` are all real,
   shareable, embeddable addresses. Keep it that way; don't move result state
   into component-only state.

---

## Architecture at a glance

```
URL (/:query)
   └─ AirPage  ──useAsync(getByQuery, query)──▶  getByQuery(query)
        │                                          ├─ getScenario()  → preset (no network)
        │                                          └─ geocode() + fetchAirQuality() + nowcastAqi()
        │                                                    ↓
        │                        { location, current, nowcast, blurb?, source? }
        │
        ├─ sketchData (memoized)  ─▶  <P5Sketch sketch={airParticleSketch} />   (the canvas)
        └─ <Readout>              ─▶  AQI meter + per-view legend                (the text column)
```

- **One page, three URLs, one component.** `App.jsx` routes `/`, `/:query`, and
  `*` all to `AirPage`. The catch-all keeps bad links rendering the app, not a
  404. `basename` supports subpath embeds.
- **Async is one ~30-line hook.** `lib/useAsync.js` runs `getByQuery(key)` when
  the URL key changes, with a per-key in-memory cache and
  `idle|loading|done|error` status. Pass it a **stable module-level function**
  (it is — `getByQuery`), never an inline arrow.
- **View/mode/toggles are local state**, deliberately *not* part of the fetch
  key, so flipping views never refetches.

### Data layer (`src/data/`, `src/lib/`)

| File | Responsibility |
|---|---|
| `data/airQuality.js` | `getByQuery` (the one entry point) → geocode + fetch + NowCast. Geocoding: ZIP via Zippopotam.us, city via Open-Meteo geocoder. Air data via Open-Meteo air-quality API. **All three are free, CORS-open, no API key** — the browser calls them directly, no serverless proxy. |
| `data/scenarios.js` | `SCENARIOS` presets + `getScenario(query)`. `getByQuery` checks this first, so `/cigarette` short-circuits the network and returns a `current`-shaped object plus a `blurb` and its own `source`. Values are illustrative literature figures. |
| `lib/pollutants.js` | `POLLUTANTS` (one entry per API field: range, `who` line, `legal` line, color), `PM25_LINES`, `exceedance(def,value,mode)`, `aqiCategory(aqi)`. The WHO/legal gap lives here. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juweek/air-diagram](https://github.com/juweek/air-diagram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
