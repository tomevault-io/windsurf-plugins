---
trigger: always_on
description: Context and rules for AI agents working in this repo. Read this before editing.
---

# AGENTS.md

Context and rules for AI agents working in this repo. Read this before editing.

## What this is

A KDE Plasma 6 plasmoid inspired by a classic skeuomorphic flip clock. The visual
design is rebuilt from QML geometry and gradients.

Target platform is Plasma 6.7 / Qt 6.11 on Arch. `README.md` is the user-facing
doc; this file is the agent-facing one.

## Hard rules

1. **Never redistribute reference assets.** `reference/` is local and gitignored.
   Do not copy its PNGs into `package/`, embed them as data URIs, or commit them.
   Measuring them to derive a colour, coordinate, or gradient stop is fine.
2. **Never trace the original glyph outlines.** Shipping traced letterforms is
   shipping the typeface. Digits use a bundled OFL font chosen by measurement
   (`tools/fontscore.py`). If you change fonts, re-run that tool and update the
   calibration constants in `Style.js`.
3. **`FlipClock.qml` must not import anything from `org.kde.plasma`.** The render
   harness loads it under bare `qml6`; a Plasma import breaks pixel-diffing. All
   Plasma coupling lives in `main.qml`.
4. **Do not set `clip: true` anywhere in the card hierarchy.** `RectangularGlow`
   deliberately draws outside its own bounds, so any clipping ancestor silently
   eats the card shadows. The digits are cropped at the crease by the
   `ShaderEffectSource` texture bound instead.
5. **Measure, don't eyeball.** Every geometric or colour change should be checked
   with `tools/pixdiff.py`. Claims like "looks right" are not acceptable evidence
   in this repo — the whole point is measured fidelity.

## Architecture

```
package/contents/ui/
  main.qml          Plasma wrapper: org.kde.plasma.clock source + config bindings
  FlipClock.qml     the whole 996x566 scene; plain QtQuick, no Plasma imports
  FlipCard.qml      one card: shadow stack, 4 flip layers, perspective, animation
  CardFace.qml      one flap: stacked-deck bezel, face gradient, crease shading
  GlossyDigits.qml  digits, gradient-filled via OpacityMask, cropped to one flap
  Style.js          .pragma library — every measured constant, in reference units
tools/
  snap.sh, Snap.qml   render a component to PNG at an exact size
  pixdiff.py          rebuild the reference frame and diff against a render
  fontscore.py        score candidate digit fonts against the measured glyphs
  probes/             fixed-state scenes for the harness (not shipped)
```

### Coordinate system

Everything is in **reference units**: the 996x566 design space.
Consumers scale by `u = min(width/996, height/566)`.

**Never wrap a fixed-size item in `Item { scale: }`** — that rasterises text once
at base size and then magnifies it. Derive every dimension from `u` instead.

`Style.js` holds scalars (geometry, timings, font calibration). Gradient stops
live as literal `GradientStop`s in the QML that paints them — paint belongs with
the painting, and it keeps one source of truth per value.

### What the original actually does

Not the usual flip-clock design, and easy to get wrong:

- **Two cards, not four digits.** Both hour digits are on one panel, both minute
  digits on another. The hour card flips on the hour; the minute card flips on
  the minute.
- **Four layers per card**: static upper = NEW, static lower = OLD, falling upper
  = OLD rotating about its bottom edge, arriving lower = NEW about its top edge.
- **Two-phase flip, `Easing.OutBounce` on both phases.** 1000 ms hours,
  800 ms minutes; the second flap is held at +90° through phase one.
- **The digit gloss restarts at the crease** so each half is lit independently.
  This is the detail that makes it read as a physical card — do not "simplify" it
  into one continuous gradient.
- The date and alarm strips on the lower third of each card do **not** flip.

## Traps that will silently waste your time

These are all verified on this machine, and each one cost real debugging.

- **`QT_QPA_PLATFORM=offscreen` cannot render this.** It runs, exits 0 and writes
  a valid PNG — but every `ShaderEffect` renders *nothing*, so the gradient-filled
  digits and all card shadows silently vanish and a diff looks catastrophically
  wrong for entirely the wrong reason. `tools/snap.sh` renders on the live
  session. The software backend is worse: no `ShaderEffect` implementation at all.
  Never "fix" a blank render by switching backends.
- **`ffmpeg -pix_fmt rgb24` drops alpha, it does not composite it.** Comparing a
  translucent render against a flattened reference this way produces completely
  bogus numbers. Flatten explicitly over a known background, as `pixdiff.py` does.
- **`MultiEffect { maskSource }` destroys text antialiasing.** It runs the mask
  through a `smoothstep` band, hard-stepping glyph edges. Use Qt5Compat
  `OpacityMask`, which is an exact `As * Am` multiply. (`qt6-5compat` is a hard
  dependency of `plasma-workspace`, so it is always present.)
- **A `GradientStop`'s `parent` is the `Gradient`, not the `Rectangle`.**
  `position: parent.someProperty` silently evaluates to `undefined` → 0 and the
  gradient collapses. Address the delegate through its `id`.
- **`font.pixelSize` is an `int`.** `270 * u` truncates; use `Math.round()`.
- **Qt5Compat effects sample source and mask in the *effect's* normalised

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pruefsumme/plasma-flipclock](https://github.com/pruefsumme/plasma-flipclock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
