---
trigger: always_on
description: These are sacred. Subagents reviewing this file MUST verify their edit does
---

# AGENTS.md — Hard guardrails for any AI agent editing this repo

## Operator-visible behaviour that MUST hold

These are sacred. Subagents reviewing this file MUST verify their edit does
not regress any of them. If unsure, leave the relevant code path alone.

### Icons

- **Every aircraft and vessel renders as its category SVG**, never as a bare
  Cesium `point`/dot, never as a blue circle. The category dispatch lives in
  `apps/web/src/globe/adapters/styles.ts` (`aircraftStyle`, `vesselStyle`).
- Aircraft categories (with their colors):
  - airliner — `#facc15`
  - private  — `#2dd4bf`
  - helicopter — `#c084fc`
  - glider — `#93c5fd`
  - military — `#f59e0b`
  - emergency squawk — `#ef4444`, pulsing
- Vessel categories (with their colors):
  - cargo — `#14b8a6`
  - tanker — `#d97706`
  - fishing — `#5eead4`
  - passenger — `#38bdf8`
  - military — `#f59e0b`
  - sailing — `#a5f3fc`
  - pleasure — `#4ade80`
  - tug — `#c084fc`
  - SAR — `#ef4444`
  - dark-vessel candidate — `#ef4444`, diamond
- Aircraft icons rotate via `track_deg` → `-Cesium.Math.toRadians(track_deg)`.
- Vessel icons rotate via `cog` (or `heading` fallback).
- Selection magenta polyline `#d946ef` width 4 + black outline width 6.

### Refresh smoothness

- **Aircraft and vessels must update in place — never disappear and reappear**.
  `PollGeoJsonAdapter` uses upsert-by-id (`getById` → update billboard image /
  rotation / position), NOT `removeAll() + add()`. Any change that re-creates
  entities on every poll is a regression and must be reverted.
- **Aircraft TELEPORT to each fix (operator request 2026-06-21).** Each poll snaps
  the aircraft straight to its newest REAL reported position via
  `ConstantPositionProperty` — no interpolation, no glide — so the map shows live
  ADS-B truth instantly. The operator explicitly chose this over the prior glide,
  with full knowledge it had been rejected twice before (see memory
  `adsb-motion-glide-to-fix`). Do NOT "fix the jump" back to a glide.
- **NEVER synthesize/predict aircraft motion — real observed fixes only.** Teleport
  shows ONLY real fixes; do NOT add interpolation, forward-extrapolation, or
  dead-reckoning to "smooth" it — that re-introduces the fake motion the operator
  rejected. The glide model (`upsertAircraftSamples`) was REMOVED in the teleport
  change; reverting to glide is a `git` revert, not a rewrite. VESSELS still glide
  via `SampledPositionProperty` + `LinearApproximation` (slow movers) — do not
  change that. Aircraft smoothness, if ever wanted again, comes from delivering
  REAL fixes faster + steadier (backend cadence, feed freshness), never from
  inventing positions.
- `requestRenderMode: true` must stay on, BUT `maximumRenderTimeChange: 0`
  (GlobeCanvas viewer opts) so the scene re-renders every frame the simulation
  clock advances — that is what makes `SampledPositionProperty` interpolation
  play SMOOTHLY between fixes instead of hopping once per poll (the "teleport"
  report). When the timeline is paused (`shouldAnimate` false) the clock is
  frozen, nothing changes, and the scene idles — so requestRenderMode still
  saves GPU. Do not set `maximumRenderTimeChange` back to `Infinity`. Follow
  (`camera.ts`) flips `requestRenderMode` off for its duration and restores it.
- **World-view decimation MUST be STABLE across polls.** At near-global zoom the
  frontend asks `/api/adsb/global?limit=20000` (no bbox); `viewport_filter`
  (`routes/adsb.py`) serves the full union (a ~13k snapshot ships WHOLE — the
  operator wants the real count, not a 4000 sample) and only decimates if the union
  exceeds 20000. When it does, it keeps a deterministic subset keyed by
  `md5(feature id)` (live tier — non-`opensky` source — first). It must
  NOT use a positional stride (`feats[int(i*stride)]`): the snapshot's order and
  count shift every refresh, so a stride resampled a DIFFERENT 4000 each poll →
  the upsert-by-id frontend churned ~half its entities every second (measured
  112% id churn / 2.5 s), which RESET the motion model so aircraft never lived
  long enough to glide and sat frozen at world view. Never key the sort on an
  age field (`seen_pos_s`/`seen_at`) — those tick every snapshot and reintroduce
  the churn. Guarded by `tests/test_adsb_viewport_stable.py`.

### Refresh cadence

- ADS-B global: 1 s frontend poll (`registry/defaults.ts` `ttlSec: 1`), backend
  sticky snapshot on a 2 s target cycle (`_SNAPSHOT_TARGET_CYCLE_S`), and each
  fan-out is wall-clock-capped at 10 s (`_FANOUT_BUDGET_S`). The 1 s poll is
  cheap (the hot route serves the sticky snapshot in microseconds); motion
  between polls is interpolated + rendered every frame. Do not raise the poll
  above 10 s.
- **Backend is HOT at boot.** `main.py` lifespan calls `adsb_routes.start_snapshot()`
  so the refresher fills `_LATEST_SNAPSHOT` before the first browser request. Do NOT
  remove it — without it the first `/api/adsb/global` runs a 1–10 s synchronous
  fan-out under `_SNAPSHOT_BOOTSTRAP_LOCK` (the "takes seconds to start loading" stall).
- **World-view payload is pre-rendered, not per-request.** The refresher builds a
  gzipped blob of the FULL snapshot (capped at `_WORLD_LIMIT` = 20000, the route

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndrewCTF/velocity](https://github.com/AndrewCTF/velocity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
