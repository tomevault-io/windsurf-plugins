---
trigger: always_on
description: A browser synthesizer built on the [Wicki-Hayden](https://en.wikipedia.org/wiki/Wicki-Hayden_note_layout) isomorphic keyboard layout — discovered through [WickiSynth](https://www.toverlamp.org/static/wickisynth/wickisynth_lowlatency.html) by Piers Titus van der Torren, the [MIDImech](https://github.com/flipcoder/midimech) visualizer by flipcoder, and the physical [Striso board](https://www.striso.org/the-note-layout/) by Piers Titus van der Torren.
---

# GridInstruments

A browser synthesizer built on the [Wicki-Hayden](https://en.wikipedia.org/wiki/Wicki-Hayden_note_layout) isomorphic keyboard layout — discovered through [WickiSynth](https://www.toverlamp.org/static/wickisynth/wickisynth_lowlatency.html) by Piers Titus van der Torren, the [MIDImech](https://github.com/flipcoder/midimech) visualizer by flipcoder, and the physical [Striso board](https://www.striso.org/the-note-layout/) by Piers Titus van der Torren.

**[Try it live](https://gridinstruments.xyz)** · [Star on GitHub](https://github.com/zitongcharliedeng/gridinstruments) · [PolyForm NC License](LICENSE) -- open source, always free. Donations welcome but never required.

![GridInstruments screenshot](tests/xstate-graph.spec.ts-snapshots/full-page-firefox-linux.png)

---

## Mission

Make isomorphic grid keyboard layouts -- especially Wicki-Hayden and DCompose -- mainstream and accessible. Harmonic literacy for everyone: an instrument that makes music theory intuitive, runs in the browser with zero install, works on as many hardware inputs as possible, and is free forever.

We want to gamify music theory through grid layouts that make intervals and chords visually obvious. Tutorials, easy multi-hardware input support, and zero-friction web access are how we spread this instrument and harmonic literacy to as many people as possible.

---

## What It Does

- **Isomorphic grid keyboard** -- DCompose and Wicki-Hayden layouts where every chord shape is the same in every key
- **Web-first synthesizer** -- runs in any modern browser, no install, Web Audio for zero-latency sound
- **Microtonal** -- continuous tuning via a fifth-size slider, from 5-TET through 7-TET and beyond, with equal temperament reference markers
- **Multi-hardware input** -- computer keyboard, MIDI controllers, touchscreen, and MPE devices
- **Expressive playing** -- MPE support, vibrato, sustain, velocity-sensitive timbre
- **Visual feedback** -- note history waterfall, staff notation, chord detection, pitch-class colors (chromesthesia in OKLCH)
- **Continuous layout morphing** -- skew slider smoothly blends between DCompose and MidiMech geometries

---

## Controls

| Control | Action |
|---------|--------|
| Letter/number keys | Play notes |
| `Shift` hold | Vibrato |
| `Space` hold | Sustain |
| Skew slider | DCompose <-> MidiMech layout morph |
| Fifth slider | Tune the generator interval (double-click = nearest TET) |
| Volume slider | Master volume |
| Zoom slider | Key size |

---

## Credits

- **[Wicki-Hayden layout](https://en.wikipedia.org/wiki/Wicki-Hayden_note_layout)** by Kaspar Wicki and Brian Hayden -- the isomorphic keyboard layout this is built on
- **[WickiSynth](https://www.toverlamp.org/static/wickisynth/wickisynth_lowlatency.html)** by Piers Titus van der Torren -- original browser synthesizer for this layout; the gateway to finding it
- **[MIDImech](https://github.com/flipcoder/midimech)** by flipcoder -- isomorphic layout visualizer and engine
- **[Striso board](https://www.striso.org/the-note-layout/)** by Piers Titus van der Torren -- physical isomorphic instrument with the same layout
- **[isomorphic-qwerty](https://github.com/xenharmonic-devs/isomorphic-qwerty)** by Xenharmonic Devs -- keyboard coordinate library

---

## Tuning

**EDO** (Equal Division of the Octave) and **TET** (Tone Equal Temperament) are the same thing for integer tunings -- 12-EDO = 12-TET. The distinction only matters for non-octave-repeating tunings (rare). This app uses EDO consistently.

The grid is a **rank-2 pitch lattice**: the x-axis follows the circle of fifths (each step = a fifth), the y-axis follows octaves. Every isomorphic keyboard -- [Wicki-Hayden](https://en.wikipedia.org/wiki/Wicki-Hayden_note_layout), DCompose, [Striso](https://www.striso.org/the-note-layout/), MidiMech -- is a 2D slice of this lattice.

**Just intonation** lives in a 3D lattice (three generators: 2/1 octave, 3/2 fifth, 5/4 major third). The grid is a 2D projection of that 3D space. **The fifth slider selects which projection**: 700 cents = 12-TET, 701.96 cents = Pythagorean (pure fifths), 696.58 cents = meantone (pure major thirds). Commas -- like the syntonic comma (81/80, about 21.5 cents) -- measure the "curvature" that equal temperament eliminates.

**Isomorphism**: every chord has the same shape in every key. Learn one chord fingering; it works everywhere on the grid, in any key, in any tuning.

---

## Development

Everything below this heading is for AI agents and contributors -- it does not appear in the app's About dialog.

---

## Architecture

### Technical Stack

TypeScript, Vite, Canvas 2D, Web Audio, Web MIDI. Vanilla TS single-page app -- no framework. All rendering via Canvas.

### Literate Programming


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zitongcharliedeng/gridinstruments](https://github.com/zitongcharliedeng/gridinstruments) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
