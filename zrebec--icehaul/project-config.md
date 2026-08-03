---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md — Ice Haul

Guidance for Claude Code when working in this repository.

## What this is

**Ice Haul** is a ZX-Spectrum-flavoured micro-simulator of ice-road trucking. Not ETS2 — its ZX hallucination. The fantasy is *risk management*, not speed: tyre pressure, cargo balance, ice patches, wind, fuel, driver fatigue. Every metre is a small decision.

GitHub repo: `zrebec/ice-haul`. Built on **[zx-kit](https://github.com/zrebec/zx-kit)** (npm `zx-kit@^0.21.0`).

## Canvas & resolution

- **Game pixels: 256 × 192** (32 × 24 cells at `CELL=8`). Pure ZX Spectrum native.
- Initialise with `setupCanvas(canvas, 4, 256, 192)` → 1024 × 768 CSS px.
- Integer scaling only. Never alter `CELL` or the palette in `zx-kit/src/palette.ts`.
- **Colour clash is a feature, not a bug.** Use `drawBitmapAttrs` with `AttrMap` (per 8×8 cell ink/paper) so two adjacent palette colours visibly bleed across a sprite — that's the look.
- Palette is the 15 hex values in `C` from `zx-kit/src/palette.ts`. No other colours, ever.

## HUD layout target (matches `first_impression.png`)

```
┌────────────────────────────────┐ rows 0–3   (status bar, 4 cells / 32 px)
│SCORE 002350       DIST 12.3km │
│                               │
│TIME 02:47       ICE AHEAD     │
│                               │
├────────────────────────────────┤
│                                │ rows 4–14  (drive viewport, 11 cells / 88 px)
│     <pseudo-3D road scene>     │   sky ~25%, road ~75%
│                                │
├────────────────────────────────┤
│E▮▮▮▮F   │    ◯ RPM    │ FREE  │ rows 15–23 (HUD, 9 cells / 72 px)
│RPM ▮▮▮▮▮│   RPM 0850  │ DRIVE │ 3 equal panels: drivetrain | tach+speed | mission
│GEAR 3/5 │   SPD  47    │       │
│GRIP ▮▮▮▮│             │       │
└────────────────────────────────┘
   85 px      85 px     86 px
```

HUD has **3 equal-width panels** (256/3). Left ("drivetrain") panel, top→bottom: **FUEL** bar, **RPM** bar, **GEAR** (current/total), **GRIP** bar — with short labels. Centre ("tachometer") panel: an **rpm dial** (needle = real engine revs, reddens at redline) plus a numeric **RPM** (real revs) and numeric **SPD** (km/h). Right: mission info placeholder. Status bar is **4 rows** — no "ICE TRUCKER" title.

> The centre panel was the **speed dial** until the lugging rework — it became a **tachometer** (owner idea 2.5, the "tach dial + prominent speed number" middle ground) so you can read real revs and see when you're lugging a too-tall gear. The left **RPM bar** now shows the raw ratio and CAN drop to 0 bars when lugging.

| Widget | zx-kit function |
|--------|-----------------|
| TACH | `drawDial` value = real rpm (`rpm × RPM_DISPLAY_REDLINE`), `min 0 max 2600`, `ticks: 5`. Needle `B_RED` at/above `REDLINE_RPM`, else `B_YELLOW`. Numeric RPM + SPD via `drawText`. |
| FUEL | `drawSegmentedBar` (horizontal). E in `B_RED`, F in `B_GREEN`, segments `B_YELLOW`. |
| RPM bar | `drawSegmentedBar` (horizontal, 7 seg). 3-stop gradient `[B_GREEN, B_YELLOW, B_RED]`; drops to 0 bars when lugging. |
| GEAR | `drawText` — current gear in `B_CYAN` (`B_RED` flash on a synchro-refused shift), `/N` total in `WHITE`. |
| GRIP | `drawSegmentedBar` (horizontal, single 6-seg bar). 3-stop gradient `[B_RED, B_YELLOW, B_GREEN]`. |
| Star field | Hand-plotted `STAR_POSITIONS` table (~17 points for compact sky). |
| Warning text `ICE AHEAD` | `drawTextCentered` + blink toggle in drive scene. |
| Kerb stripes | Pole-Position-style alternating `B_WHITE`/`B_YELLOW`, perspective-scaled width. |

## Gameplay model (decided)

- **Progression**: campaign + save. A series of seasonal routes, increasing difficulty, unlockable rigs/routes. Highscore table per route.
- **Decisions UX**: hybrid.
  - **Pit-stop scene** between segments — large decisions: tyre choice, cargo lashing, fuel fill, rest.
  - **Real-time in-drive** — small switches: heater, wipers, headlight high-beam, CB radio. Bound to number keys / gamepad shoulders. The risk is taking eyes off the road.
- **Audio**: AY engine drone modulated by RPM during drive + beeper SFX (warning beep, gear shift, ice crack, low-fuel chirp). AY tracker music for menus/intro/pit-stop only.

## Module layout (target)

```
src/
  main.ts            entry: setupCanvas, scene manager bootstrap, audio init on first input
  config.ts          LANGUAGE_CODE, debug flags, constants
  strings.ts         English UI strings (default locale)
  strings.sk.ts      Slovak (gitignored if needed; pickLocale via zx-kit i18n)
  save.ts            wires zx-kit save profile — campaign state, unlocks, highscores
  scenes/
    drive.ts         the main driving scene ✓ (phase 1)
    intro.ts         title + press-key                (phase ≥6)
    menu.ts          new game / continue / options    (phase ≥11)
    pitstop.ts       between-segment decisions        (phase 6)
    pause.ts         pushed on top of drive           (phase ≥6)
    gameover.ts                                       (phase 5)
  game/
    vehicle.ts       ✓ throttle/brake/steer + grip-scaled lateral physics
    road.ts          ✓ deterministic surface lookup (asphalt/ice)
  render/
    road3d.ts        ✓ scrolling pseudo-3D road with per-scanline surface
    truck.ts         ✓ rear-view player truck sprite

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zrebec/icehaul](https://github.com/zrebec/icehaul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
