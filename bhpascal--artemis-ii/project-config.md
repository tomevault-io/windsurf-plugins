---
trigger: always_on
description: A Bret Victor-style "explorable explanation" of the orbital mechanics behind NASA's Artemis II free-return lunar trajectory. The mission launched April 1, 2026 and is currently in progress. The page teaches orbital mechanics through interactive visualizations where the reader manipulates parameters and watches trajectories change in real time.
---

# CLAUDE.md — Artemis II Explorable Explanation

## What This Is

A Bret Victor-style "explorable explanation" of the orbital mechanics behind NASA's Artemis II free-return lunar trajectory. The mission launched April 1, 2026 and is currently in progress. The page teaches orbital mechanics through interactive visualizations where the reader manipulates parameters and watches trajectories change in real time.

**Inspirations**: Bret Victor's [Ladder of Abstraction](https://worrydream.com/LadderOfAbstraction/), Nicky Case's explorable explanations, Wired's "5 Levels of Difficulty" video series.

**Full project spec**: `docs/project-spec.md` — contains all research data, physics requirements, section designs, and architecture decisions. Read it before starting work.

**Live data sources**: `docs/live-data-sources.md` — API endpoints for real telemetry (community orbit API, AROW, JPL Horizons, mission timeline). Already integrated into Hook section.

**Claude Web visualization reference**: `docs/claude-web-viz` — comprehensive CR3BP analysis including the Arenstorf orbit approach (see "Current Status" below).

**Flyby rewrite spec**: `docs/flyby-rewrite-spec.md` — HISTORICAL. The patched-conic solver was replaced by a CR3BP integrator (`src/physics/cr3bp.ts`). This spec documents the debugging journey but the code it describes no longer exists.

## Current Status (as of 2026-04-08)

**What's working:**
- Sections 1-4 render with CR3BP trajectories
- **Free Return section uses the Arenstorf periodic orbit at real μ = 0.012277471** — perfect closed figure-8, no mass enhancement. Slider controls velocity perturbation (±30 m/s); at 0 it's periodic, at 10 m/s the orbit misses closure by 80% of D_MOON (chaos demo).
- **Real Mission section uses real JPL Horizons ephemeris** for Orion (ID -1024) — actual spacecraft state vectors at 30-minute intervals, transformed into the 2D co-rotating frame. Flyby data matches reality: 413,005 km from Earth, 8,520 km from Moon, 0.42 km/s at T+5d. The Arenstorf orbit still shows as the figure-8 ghost background.
- Live telemetry in Hook section from `artemis.cdnspace.ca/api/orbit` (updates every 60s)
- Countdown to lunar closest approach
- Behind the Scenes page at `/behind-the-scenes`
- Physics test suite: `npm test` runs 22 vitest tests. Run these after any `cr3bp.ts` change.

**Refreshing ephemeris data:**
`npm run fetch-ephemeris` runs `scripts/fetch-ephemeris.ts`, which hits the JPL Horizons API for both Orion (-1024) and Moon (301), parses the Vectors table, transforms into the 2D co-rotating frame, and writes `src/data/orion-trajectory.json`. The JSON is committed to git — builds do not hit Horizons. Re-run manually when fresh data is wanted (e.g., post-splashdown). The pre-Horizons phase (first ~3.4 hours) still uses a synthetic HEO orbit approximation because Horizons has no data before then.

**Architecture note — CR3BP vs real data:**
- `propagateArenstorf(vyPerturbation)` at `MU_REAL` — powers the interactive Free Return section
- `propagate(injectionDv, ...)` at `MU_CR3BP = 0.15` — **no longer used by any section**; retained only for the legacy test script. Can be removed after verifying no other consumers.
- `computeArtemisTrajectory()` in `trajectory.ts` now reads bundled Horizons JSON instead of running a CR3BP integration.

**What's next:**
- **Inertial frame toggle**: Commented out in Free Return — needs CR3BP→inertial coordinate rotation. Transform formulas are in `docs/claude-web-viz`. Now that the orbit itself is clean, this is mostly a coordinate-math task.
- **Behind the Scenes page**: Add the μ enhancement debugging story and the Arenstorf transition to the narrative. Plus now the "real data replaces simulation" story for Section 5.
- **Cleanup**: Delete `MU_CR3BP` and the LEO `propagate()` function once confirmed nothing else depends on them.

## Tech Stack

- **React + TypeScript** — component-based, type-safe
- **Vite** — fast dev server, simple build, static output
- **Modern CSS** — CSS Modules or vanilla CSS, no Tailwind (Tufte typography needs precise control)
- **HTML5 Canvas** — for orbital visualizations (not SVG — too slow for thousands of path points)
- **No heavy dependencies** — no MathJax/KaTeX (hand-crafted HTML math so equation terms can be scrubable), no physics libraries (we write the orbital mechanics)
- **Deploy**: Vercel static site from `dist/`

## Design Philosophy

### Tufte Typography
- ET Book font family (load from CDN: `edwardtufte.github.io/tufte-css/`)
- ~740px content column, generous right margin for sidenotes
- Sidenotes, not footnotes. On mobile (<760px), sidenotes collapse to toggleable inline elements
- Background: `#fffff8`. Text: `#111`. Links: understated
- Headings use size and spacing, not bold (Tufte convention)
- Math rendered as styled HTML (`<var>`, `<sup>`, flex-based fractions), not images or library output

### Interactive Elements

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bhpascal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
