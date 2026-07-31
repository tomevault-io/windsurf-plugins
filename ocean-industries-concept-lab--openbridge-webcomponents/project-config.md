---
trigger: always_on
description: generates a box (`observeInnerBox()` in radial-frame.ts observes the
---


# GitHub Copilot Custom Instructions

## Path-Specific Instructions for Watch & Radial Instruments

These instructions apply to the circular/radial watch-based instrument system, including the core `obc-watch` renderer and all navigation instruments that use it.

> **⚠️ IMPORTANT: Interconnected Components**
>
> All components in this system are **tightly interconnected** and share the same rendering core:
>
> - `watch.ts` ↔ `instrument-radial.ts` ↔ `compass.ts` ↔ `heading.ts` ↔ `rudder.ts` ↔ `wind.ts` ↔ `speed-gauge.ts` ↔ `gauge-radial.ts` ↔ `azimuth-thruster.ts` ↔ `roll.ts` ↔ `rot-sector.ts`
>
> **When implementing a new feature or changing existing behavior:**
>
> 1. **All rendering logic should live in `watch.ts`** - it is the single source of truth for circular instrument rendering
> 2. Changes to `watch.ts` affect ALL instruments that use it
> 3. If adding a new visual element, add it to `watch.ts` as a configurable option, not to individual instruments
> 4. Navigation instruments are thin wrappers that configure `obc-watch` and add domain-specific overlays
> 5. `instrument-radial.ts` is a reusable building block that wraps `obc-watch` for generic gauge use cases
>
> **Before completing any change, verify:**
>
> - [ ] `watch.ts` has the core rendering logic
> - [ ] Helper modules (`tickmark.ts`, `advice.ts`, `label.ts`, etc.) are updated if needed
> - [ ] All consuming instruments still render correctly
> - [ ] ViewBox coordination is maintained across layers
> - [ ] Responsive scaling works at different sizes

## Architecture Overview

The watch-based instrument system follows a **core renderer + thin wrapper** pattern:

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              watch.ts (obc-watch)                               │
│                    (Core SVG renderer - ALL logic lives here)                   │
│                                                                                 │
│  Renders:                                                                       │
│  • Circular rings (single/double/doubleThin/triple)                            │
│  • Setpoint indicator (triangle marker)                                        │
│  • Tickmarks (primary, secondary, main, textOnly)                              │
│  • Advices (caution/alert zones with patterns)                                 │
│  • Bar areas (filled arc segments)                                             │
│  • Needles (short bar indicators)                                              │
│  • Labels (N, E, S, W compass labels)                                          │
│  • Vessel images                                                               │
│  • Wind/current indicators                                                     │
│  • North arrow, crosshair, starboard/port indicators                           │
│                                                                                 │
│  Imports helper modules:                                                        │
│  • tickmark.ts - tickmark rendering & positioning                              │
│  • advice.ts - advice/caution zone rendering                                   │
│  • label.ts - compass label rendering                                          │
│  • vessel.ts - vessel image SVGs                                               │
│  • environment.ts - wind/current symbols                                       │
│  • setpoint.ts - setpoint indicator rendering                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
                    ▼                   ▼                   ▼
    ┌───────────────────────┐ ┌─────────────────┐ ┌─────────────────────────────┐
    │  instrument-radial.ts │ │  compass.ts     │ │  Other instruments:         │
    │  (Generic building    │ │  heading.ts     │ │  rudder.ts, wind.ts,       │
    │   block for gauges)   │ │  (Full-featured │ │  speed-gauge.ts, roll.ts,  │
    │                       │ │   compasses)    │ │  azimuth-thruster.ts, etc. │
    └───────────┬───────────┘ └────────┬────────┘ └──────────────┬──────────────┘
                │                      │                         │
                ▼                      │                         │
    ┌───────────────────────┐          │                         │
    │  gauge-radial.ts      │          │                         │
    │  rot-sector.ts        │          │                         │
    │  (Thin wrappers)      │          │                         │
    └───────────────────────┘          │                         │
                                       ▼                         ▼
                              ┌─────────────────────────────────────────────────┐
                              │  All instruments use <obc-watch> + overlay SVG  │
                              │  with matched viewBox for layer alignment       │
                              └─────────────────────────────────────────────────┘
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ocean-Industries-Concept-Lab/openbridge-webcomponents](https://github.com/Ocean-Industries-Concept-Lab/openbridge-webcomponents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
