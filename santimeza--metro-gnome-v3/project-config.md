---
trigger: always_on
description: This is the final build spec, ready to hand to an agentic coding tool. All open decisions from planning are resolved (see status log at the bottom). Treat this as the source of truth for the rebuild.
---

# metro-gnome — build spec

This is the final build spec, ready to hand to an agentic coding tool. All open decisions from planning are resolved (see status log at the bottom). Treat this as the source of truth for the rebuild.

---

## 1. Project overview

A rebuild of an existing metronome site into four tools sharing one accurate audio engine:

- `/metronome` — metronome with per-beat customization
- `/strum` — strum machine with a hybrid sample/synthesis guitar engine and a song library
- `/tuner` — microphone-based tuner with a custom range
- `/timing-game` — tap-along rhythm game (ported from the existing site, lowest priority)

**Stack:** React + Vite.

**Repo history:** two earlier attempts exist (`metro-gnome` on GitHub Pages, and an unfinished `Metro-gnomeV2` React rewrite). Both drove timing with `setInterval`, which is not sample-accurate and drifts over time — this is being replaced (see below), not carried forward.

---

## 2. Shared audio engine (build this first)

Everything else depends on this. Replace `setInterval`-driven timing with a look-ahead scheduler:

- A coarse `setInterval` (~25ms) checks whether anything needs to be queued in the next ~100ms window
- Each event's actual playback time is computed as an exact `audioContext.currentTime` offset, not "whenever the timer fires"
- Any visual element (beat flash, swinging indicator, strum playhead) reads timestamps from this same schedule via `requestAnimationFrame`, rather than running its own independent timer or CSS animation

This one module underlies the metronome, the strum machine, and the timing game. The tuner does not use it (it consumes mic input rather than driving playback), but should live in the same shared audio utilities file for consistency.

---

## 3. Metronome page (`/metronome`)

**Core controls**
- BPM range 20–300: tap-tempo button, +/- steppers, slider, and direct numeric input, all kept in sync
- Time signature selector, presets only: 2/4, 3/4, 4/4, 5/4, 6/8, 7/8, 9/8, 12/8 — no custom numerator/denominator input

**Visual indicator**
- One metronome graphic centered at the top of the page (swinging needle/pendulum look), driven directly by the shared scheduler
- The row of beat cells (below) also flashes its own corresponding cell as each beat plays — top-level swing and per-beat detail visible at the same time

**Per-beat customization**
- A row of cells, one per beat in the measure. Each cell is independently set to **accent**, **normal**, or **muted/silent**, and assigned a sound from the sound palette
- The row automatically resizes to match the time signature (3/4 → 3 cells, 6/8 → 6 cells, etc.), with sensible accent defaults on beat 1 (and beat 4 for compound meters like 6/8)
- Sound palette: reuse the existing `click_1/2/3.wav` samples, plus a few generated with simple Web Audio oscillators (woodblock, cowbell, digital beep) rather than more recorded assets
- `guitarLoop.wav` from the old site is not carried forward — drop it

---

## 4. Strum machine page (`/strum`)

**Core structure**
- A chord progression: an ordered list of chords, each with a duration in beats or measures
- A strum pattern editor: a grid aligned to the time signature, each subdivision slot set to down-strum, up-strum, mute/chuck, rest, or accent
- Song sections (intro/verse/chorus/etc.), each with its own progression and pattern
- Tempo, key, and capo controls — capo/key changes transpose both displayed chord names and audio
- Swing/shuffle amount, and per-strum dynamics (accent vs ghost strum)
- Shares its tempo/clock with the metronome page's scheduler

**Rhythm pattern library**
Preset patterns, selectable per section and editable after picking:
- **Boom-chuck** — alternating bass note on beats 1 and 3 (root, then a second bass note such as the fifth or next-lowest available string), chord chuck on beats 2 and 4
- **Drone** — a genuinely sustained, bowed-string-like tone (not a decaying pluck) for a held, violin-ish character
- Standard folk/pop strum patterns (e.g. D-DU-UDU), a waltz pattern, a reggae skank, and a couple of fingerstyle-adjacent arpeggiated patterns
- Custom: build and save a pattern from the grid

**Guitar sound engine — hybrid approach**
1. **Sample layer:** a set of recorded strums covering open-position chords in the most common keys. Loaded via Vite's `import.meta.glob('/src/samples/**/*.wav')`, so any file dropped into `src/samples/` following the naming convention `{root}-{quality}_{position}_{strumDirection}.wav` (e.g. `E-major_open_down.wav`) is automatically discovered at build time — no manifest file, no registration step.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [santimeza/metro-gnome-v3](https://github.com/santimeza/metro-gnome-v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
