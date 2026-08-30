---
trigger: always_on
description: description: Sol Obscura OBS widget project — full context for AI code assistance
---

# Sol Obscura — Cursor Rules
# Place this file at .cursor/rules/sol-obscura.mdc in the project root.
# This gives Cursor full context for the Sol Obscura OBS widget codebase.

---
description: Sol Obscura OBS widget project — full context for AI code assistance
globs:
  - "**/*.html"
  - "**/*.css"
  - "**/*.js"
  - "**/*.md"
alwaysApply: true
---

## Project Overview

Sol Obscura is a Burning Man-adjacent music and art event centred on the **total solar eclipse
of 12 August 2026** in Tortosa, Catalonia, Spain.

This repository contains **OBS Browser Source widgets** — self-contained HTML files that run
inside OBS Studio as live overlays on a projected screen at the venue. They are information art
installations, not broadcast graphics. The guiding philosophy is restraint: the screen should
serve curiosity without competing with the sky.

### Key Event Facts

| Property             | Value                                              |
|----------------------|----------------------------------------------------|
| Totality start       | 20:29:51 CEST (18:29:51 UTC)                       |
| Totality duration    | 1 minute 31 seconds (91 seconds)                   |
| Sun altitude         | 4.7° above western horizon                         |
| Venue                | South of Tortosa, Ebro Delta, Catalonia, Spain     |
| Saros cycle          | 126                                                |
| Audience             | General Catalan / Spanish public, all ages         |
| Totality UTC epoch   | 2026-08-12T18:29:51Z                               |

---

## File Structure

```
./
  countdown-widget.html       # Countdown to totality — OBS top-right corner overlay
  NOAA-solar-wind-overlay.html # Live solar wind data — OBS bottom-third panel
  SDO.html                    # Auto-refreshing NASA SDO solar image — OBS corner widget
  weather-solar.html          # Live solar irradiance panel (0x15)
  weather-uv.html             # Live UV index panel (0x17)
  weather-wind-speed.html     # Live wind speed panel (0x0B)
  weather-wind-direction.html # Live wind direction panel (0x0A)
  weather-pressure.html       # Live relative pressure panel (wh25.rel)
  weather-solar-graph.html    # Live solar irradiance graph panel
  weather-banner.html         # Right-edge combined weather banner
  [future widgets here]
docs/
  sol-obscura-brand-reference.md
  Sol-Obscura-Science-Dashboard-Design-Guide.pdf
  TextandBannerBestPractices.md
.cursor/
  rules/
    sol-obscura.mdc           # This file
```

---

## OBS Scene Architecture

Widgets are loaded as **Browser Sources** in OBS Studio at 1920×1080.
Five scenes run on a timed schedule via OBS Advanced Scene Switcher:

| Scene                    | Time (CEST)         | Widgets active                                      |
|--------------------------|---------------------|-----------------------------------------------------|
| AMBIENT_COSMOS           | Aug 10–12 daytime   | countdown (top-right, small), NOAA overlay, SDO     |
| ECLIPSE_BUILD            | Aug 12 ~18:30–20:25 | countdown (centre, large), NOAA overlay             |
| TOTALITY                 | Aug 12 20:28–20:32  | Minimal text only — NO countdown, NO data overlays  |
| POST_ECLIPSE_REFLECTION  | Aug 12 20:32+       | NOAA K-index, static title card                     |
| NIGHT_AMBIENT            | Nightly 22:00–07:00 | ISS feed, science fact rotator                      |

**Scene 3 (TOTALITY) is the most important design constraint**: the screen steps back.
No urgency, no flashing, no countdown. One line of bilingual text, one SDO image, silence.

---

## Design Principles

These values must be reflected in all code, copy, and layout decisions:

- **Immediacy** — widgets invite presence; they never dominate or mediate the eclipse itself
- **Radical Inclusion** — all text is bilingual (Catalan first, Spanish second); font sizes
  readable from 4 metres; accessible contrast ratios throughout
- **Leave No Trace** — no persistent storage, no cookies, no tracking; widgets are stateless
- **Radical Respect** — no auto-playing audio; no flashing or strobing effects
- **Gifting** — all data sources are free and public; no paywalled APIs

---

## Colour Tokens

All widgets **must** use these CSS custom properties. Do not introduce ad-hoc hex values.

```css
:root {
  /* Core palette */
  --color-bg:           #0A1A1F;   /* Deep Void — primary background */
  --color-solar-gold:   #FFB81F;   /* Solar Gold — primary accent, countdown digits */
  --color-pale-gold:    #FDDC91;   /* Pale Gold — secondary labels, body text on dark */
  --color-text:         #FCF7F7;   /* Warm Off-White — reading text */
  --color-divider:      #656343;   /* Olive Shadow — borders and <hr> ONLY, never text */
  --color-grey-mid:     #A9A9A9;   /* Mid Grey — disabled / placeholder text */
  --color-grey-light:   #C1C1C1;   /* Light Grey — borders */

  /* Extended dashboard tokens */
  --color-corona:       #E05C00;   /* Corona Orange — alerts on deep-void bg only */
  --color-corona-bright:#FF7A2F;   /* Bright Corona — alerts on teal panel surfaces */
  --color-panel:        #1A4A4F;   /* Twilight Teal — panel/card surfaces */
  --color-iss-blue:     #C8D8E8;   /* ISS Blue — Earth feed and geomagnetic data */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rafterv/sol-obscura](https://github.com/rafterv/sol-obscura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
