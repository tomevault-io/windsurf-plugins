---
trigger: always_on
description: Monarch of Tone is a circuit-level emulation of the Analog Man King of Tone overdrive pedal,
---

# Monarch of Tone — Project Memory

Monarch of Tone is a circuit-level emulation of the Analog Man King of Tone overdrive pedal,
built as an AU/VST3 plugin using JUCE 8+ and chowdsp_wdf Wave Digital Filter modelling.
**Author/Company:** Leigh Pierce

The King of Tone is a **dual-channel** Bluesbreaker-derived overdrive — a 1-to-1 digital clone.
Both channels run in series and are independently bypassable. They're named by LED colour:
**Yellow** (left, stock) and **Red** (right). **Signal flow is Red → Yellow** — the real pedal
processes through Red (the fixed Hi-Gain channel) *first*, then Yellow. Yellow/Red on-screen
position is unchanged from the hardware; small **A** (Red) / **B** (Yellow) badges outside each
LED show the processing order. The Theseus Hi-Gain mod is a **fixed** part of the **Red** channel
only (not a runtime toggle); Yellow is always stock.

---

## Rule Files (read before touching any code)

@.claude/rules/circuit.md      ← circuit topology, all component values, signal flow
@.claude/rules/dsp.md          ← WDF implementation rules, API reference, diode parameters
@.claude/rules/architecture.md ← plugin structure, APVTS parameters, threading model
@.claude/rules/ui.md           ← layout, controls, colour scheme
@.claude/rules/build.md        ← CMake setup, project structure, validation gates

**Agents:**
- `dsp-validator` — invoke after implementing each DSP stage. Don't proceed on a non-PASS verdict.
- `schematic-checker` — invoke for any circuit topology question. Never guess a component value or
  connection from memory.

---

## Quick Reference

```bash
# First-time setup
git submodule update --init --recursive   # JUCE 8+, chowdsp_wdf, xsimd (optional SIMD)

# Build
cmake -B build -DCMAKE_BUILD_TYPE=Release && cmake --build build
cmake --build build --target Monarch_AU     # AU (primary) / Monarch_VST3 / Monarch_Standalone

# Code quality
clang-format -i src/**/*.{cpp,h}
```

**CMake minimum:** 3.15 | **C++ standard:** 17 | **macOS target:** 10.13+ | **Version:** 1.4.0
(single-sourced from `project(MonarchOfTone VERSION …)` in CMakeLists.txt → `MONARCH_VERSION_STRING`
→ the UI's version label; never hardcode it anywhere else)

---

## Roadmap

- **v1.0 — Project cleanup.** ✅ Done — repo/docs audited and condensed.
- **v1.1 — CPU / latency / memory optimization pass.** ✅ Done (2026-06-29).
- **v1.4 — FR / harmonic accuracy pass.** ✅ Done (2026-07-29). Full findings and the
  worked P0–P10 plan → `analysis/FR_THD_AUDIT.md`. The ordered plan is finished; all residual
  gaps are accepted and documented.
- **v1.5 — CPU pass + the ADAA identity-region droop.** ✅ Done (2026-07-30). Full
  findings, span split, every lever's measured cost, rejected levers with evidence, and
  measurement protocol → `analysis/CPU_AUDIT.md`. Steps 1–5 are shipped.

**Audit docs:** `analysis/FR_THD_AUDIT.md` (v1.4 accuracy), `analysis/CPU_AUDIT.md` (v1.5 CPU),
`analysis/MID_EQ_AUDIT.md` (mid-EQ correction audit), `analysis/VALIDATION_REPORT.md`
(per-capture null breakdown).

---

## Repository State

**Engine + UI complete and validated; matched to NAM captures of a real King of Tone.** All DSP
stages, `MonarchChannel`, `processBlock`, and oversampling are done (build.md Validation Gates,
all PASS, auval PASS). The UI is complete: peripheral shared-look shell (side panels, OS strip,
resizable window) + the purple/gold `PedalFace`. Factory presets (5) ship via the host's native
preset browser. Supply-voltage mod (9/12/18V) and rail-saturation ADAA are in. CI/CD
(`.github/workflows/`), cross-platform VST3, and per-platform installers (`installer/`) — see README.

**Calibration result:** the plugin nulls against 44 NAM captures (drive G2–G10, tone T2–T8,
Clean/OD/Dist) at **−8.6 to −27.0 dB, median −23.4 dB**, and is at **−22.0 dB or better on every
capture from G2 to G7**. Per-mode at G5 T5: Clean **−23.9**, OD **−24.8**, Dist **−23.1**. The
match is shallower only at very high drive (G8–G10) — an accepted device-physics / capture-aliasing
residual, not a topology error. The 44 captures (`analysis/pedal_export2/`, 842 MB) are
**local-only, gitignored**.

### Performance (v1.5 step 5, 2026-07-30)

`PerfBenchmark` — one CPU core, 48 kHz, both channels active in series, stereo:

| OS | Boost | Overdrive | Distortion |
|----|-------|-----------|------------|
| 1x | 2.1% | 3.8% | 2.8% |
| 2x | 2.8% | 5.3% | 3.6% |
| 4x | 4.8% | 9.8% | 6.5% |
| 8x | 8.6% | 18.8% | 12.1% |
| render | — | — | 11.1% |

### Accepted residuals

- **Sub-32 Hz shortfall** (~2 dB at 20 Hz). Not a topology error; the pedal's phase *leads*
  below ~32 Hz, so no minimum-phase filter matches. P1 tried, P8 narrowed it significantly.
  Documented in `FR_THD_AUDIT.md` P1/P8.
- **G8–G10 clip-path behaviour** — the OD ceiling over-corrects at G10 (wrong-signed by
  construction: the pedal's OD compresses *less* at G10 while a level-keyed ceiling bites
  *more*). About half is target-side noise (the pedal's OD curve varies 1.19 dB across TONE,
  a knob that categorically cannot affect it). P10 step 3 measured the target's floor and
  accepted the residual. See `FR_THD_AUDIT.md` P9/P10.
- **HF harmonic difference above 8 kHz** (tone-stage rolloff; the captures' own 4–6 kHz energy
  is partly NAM aliasing).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tehguitarist/Monarch-of-Tone](https://github.com/tehguitarist/Monarch-of-Tone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
