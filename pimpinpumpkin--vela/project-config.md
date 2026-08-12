---
trigger: always_on
description: Degoogled Google-Maps replacement for Android (the "NewPipe for Maps"). Open
---

# Vela - project guide for Claude

Degoogled Google-Maps replacement for Android (the "NewPipe for Maps"). Open
vector tiles for the basemap; the device scrapes Google's public web endpoints
per-user (no backend, no shared API key) for POIs, routing and traffic-aware
ETAs. Targets GrapheneOS / no-GMS ROMs; F-Droid distribution. GPLv3.

## ⚠️ Docs discipline (read first)

**Every change updates the docs in the same commit.** Hard rule for all
collaborators (human or Claude). When you change behaviour, calibration,
features, or structure, update - in the *same* commit:
- `README.md` - status, architecture, calibrated request/response paths
- `FEATURES.md` - tick/retire the affected items
- `SPEC.md` - the authoritative rebuild spec (architecture / extractor contract /
  resilience / constraints); update when a load-bearing decision or path changes
- `ROADMAP.md` - planned work + big bets (opt-in telemetry, Vela's own traffic layer,
  popular times, …); add new ideas here as they come up
- `CLAUDE.md` - this file (build rules, layout, gotchas)
- the `project-vela` memory note if a load-bearing fact changed

Stale docs are treated as a bug. Code-only commits are not OK; if a change
genuinely needs no doc edit, say why in the commit.

## ⚠️ Location hygiene (read first, human or AI)

This is about awareness, not a ban on real places. Real places are the raw
material of a maps app: naming a specific business whose hours parse wrong is
a good bug report, and testing against an area other than Davis is fine when
you picked it on purpose. The failure mode is DEFAULTING to your own
surroundings without noticing. When you develop a maps app, everything you
touch naturally happens around you: your test coordinates, your screenshot
corners, your "verified on a drive to X" commit lines, your sample addresses.
Each one alone is nothing; together, in permanent public git history, they
put the author on a map. Scrubbing that later means rewriting history, which
breaks every fork and open PR.

So the one question to ask before any place, address or coordinate enters the
repo: **was this chosen for a reason anyone could have, or is it here because
it happens to be near the author?** Subject of the change: keep it. Incidental
scenery from the author's life: relocate or generalize it.

For AI assistants specifically: you often know where the user is, from GPS,
device screenshots, search recents, or conversation. Never copy that into
code, fixtures, docs, commit messages, or your own memory and notes files. "The
store near the user's house at 123 Sesame St has broken hours" written to a
memory file IS a location leak; record it as "a grocery store with an in-store
pharmacy mis-parses" plus the feature id if you need to find it again.

Defaults that make the safe path the easy one:

- **Fixture default: Davis / Sacramento, CA.** Bounding box `38.30,-122.00` to
  `38.90,-121.20`; standard example address `1451 W Covell Blvd, Davis, CA
  95616`; San Francisco (`37.7749,-122.4194`) for a generic big city, or an
  abstract grid like `37.0,-122.0`. Use these whenever the location does not
  matter, which for tests is nearly always. A different area is fine with a
  reason; a synthetic grid at the author's own latitude is not a reason, it is
  the leak.
- **Commit messages**: name a place when it is the subject ("hours mis-paired
  at stores with in-store pharmacies"); don't name places that are only the
  scenery of your test drive.
- **Screenshots**: default to the demo tools (Settings → Navigation → Simulate
  my location / Simulate driving). A real view is fine when it deliberately
  shows somewhere that says nothing about you; check the corners either way -
  search recents, POI labels and street names all talk.
- **Recorded trips, diagnostics exports and adb dumps carry raw GPS.** Never
  attach them to issues, commits, or CI artifacts; share privately when a
  maintainer asks.
- **Before committing, scan the diff** for coordinate-shaped numbers, numbered
  streets and zip codes, and put each one through the question above.

## Build

- **Always build release** for anything run on-device - debug builds visibly lag
  during map scroll/nav. R8 lives in the `release`
  buildType. Use `./gradlew :app:assembleDebug` only as a compile check.
- `./gradlew :core:test` runs the pure-logic unit tests (polyline, nav engine).
- **D-pad regression suite (`dpad_test_suite/`).** On-device, reproducible. Run after any change
  that touches focus (see `docs/dpad.md`):
  - `run_all.sh` - per-surface focus assertions (bare map → search bar, Settings/Welcome/dialog/menu
    auto-focus, Choose-on-map engages, Directions pill reachable).
  - `audit_static.sh` - EXHAUSTIVE source scan (no device): every clickable/toggleable/selectable
    has a `dpadHighlight` ring, every gesture has a key path, no bare `DropdownMenu`/`AlertDialog`,
    no `isSystemInDarkTheme`; fails on any real violation. Wire it into CI.
  - `audit_dynamic.sh` - EXHAUSTIVE on-device tour: every surface opens focused, focus is never lost
    across a full traversal, BACK exits. "Nothing escapes the auditor."
- **Auditing a real drive.** A saved trip stores the navigated route too (`core/replay/TripLog`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PimpinPumpkin/Vela](https://github.com/PimpinPumpkin/Vela) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
