---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A tool that **auto-calibrates a 3D printer's XY shrinkage and skew from a flatbed scan** of a printed
calibration coupon: no manual caliper measurements. The user prints `calibration_coupon.scad` (an open
lattice of measurement rings), scans it, and the software reads the geometry with OpenCV.js and emits
ready-to-paste firmware/slicer corrections.

The measurement principle: ring **centres** give true X/Y scale and skew (centres are immune to
over/under-extrusion, because extrusion changes a ring's wall width, not its centre). The correction math
mirrors the Vector 3D "Califlower" calculator (Klipper `SET_SKEW`, Marlin `XY_SKEW_FACTOR`/steps-per-mm,
Orca/Super shrinkage %, RRF `M556`).

Orientation is automatic. The coupon's origin-corner ring **and its +X neighbour** are printed SOLID (no
hole): a two-ring marker the software reads. `origin → neighbour` is the coupon's +X, which resolves
rotation AND mirror-flip with no manual input (see "Coupon & orientation" below).

## The app: a Vue 3 web app

The app is a plain web app under `web/` (Vue 3 + TypeScript + Vite + Vuetify). **Web is the only target.**
The CV measurement pipeline is ported to TypeScript and runs in a **Web Worker** using **OpenCV.js**, so
analysis is off the main thread (the page never freezes), needs no cross-origin-isolation headers (works on
GitHub Pages), and is fast (V8 JIT + an optimized OpenCV.js build). Native `<input type=file>` and
`<input type=number>` mean there is no soft-keyboard, touch-stepper, or iOS file-input workaround to carry.

Commands (run inside `web/`):

```bash
npm install
npm run dev       # Vite dev server at http://localhost:5173/
npm run build     # vue-tsc typecheck + production build to web/dist
npm test          # Vitest: engine unit tests + fixture-backed CV tests
npm run e2e       # Playwright end-to-end over the real scans in web/e2e/fixtures
```

Structure:

- **`web/src/engine/`**: the framework-agnostic measurement engine (no Vue, no DOM assumptions beyond what
  OpenCV.js needs). Each function takes the loaded `cv` instance as a parameter, so OpenCV.js stays out of
  the main bundle (it lives in the worker chunk, loaded on first analysis) and tests can inject it. Stages:
  `ringDetector`, `gridMapper`, `affineSolver`, `couponAnalyzer`, `scanCombiner`, `cardEdgeMeasurer`,
  `overlayRenderer`, `correctionFormatter`, plus `types`, `opencv` (loader), `imageData`, and shared
  helpers `math`/`cvUtils`.
- **`web/src/worker/`**: a Comlink Web Worker (`analysis.worker.ts`) exposing `analyzeTwoScans` and
  `measureCardScan`; `decode.ts` decodes image bytes with `createImageBitmap` + `OffscreenCanvas` and
  renders overlays back as `ImageBitmap`. `web/src/workerClient.ts` is the only thing the UI calls for CV.
- **`web/src/components/`**: thin Vue pages (`ScanPage`, `CalibrationPage`, `ResultsPage`) plus the guide
  diagrams and controls, over Pinia stores in `web/src/stores/` (`useApp` for navigation + payload,
  `useCalibration` for the localStorage-backed scanner calibration).
- **Tests**: `web/tests/` (Vitest engine + fixture CV tests, with `tests/helpers/cv.ts` and
  `tests/fixtures/TestData_2solid.png`) and `web/e2e/` (Playwright over the real scans in
  `web/e2e/fixtures/`).

Absolute scale needs a known px/mm (scanner DPI is rarely exact), so the app measures a standard ISO/IEC
7810 plastic card (`cardEdgeMeasurer`) to learn the true px/mm; without it, only anisotropy and skew are
meaningful. The card is measured along its LONG side only: the short side reads through the lid-shadow
zone and is banned as a reference. The scanner's transport axis also carries low-frequency mechanical
waviness (about 0.1 mm on real units), so fine lateral measurements (ringing wiggles) are only read along
the sensor-row axis; flows that need both directions scan the part twice, a quarter turn apart.

A scanner's scale error can be per-axis: a CCD scanner is mis-scaled along its sensor axis but accurate
along the carriage axis, so a calibration is a `ScaleReference` (a scalar px/mm, or a per-axis pair for
CCD), never a bare number. Every flow that converts scan pixels to millimetres must take the
`ScaleReference` from `scaleReferenceAtDpi` (the only exported way to price a stored calibration) and
convert along its actual measurement direction via `referenceAlongDirection`; a new flow that pockets a
scalar px/mm reintroduces the wrong-axis bug on CCD scanners. Each flow's render-recovery tests should
include one case with a deliberately wrong figure on the unused axis to prove it cannot leak in (see the
quarter-turned coupon case in `emAnalyzer.spec.ts`).

Two durable gotchas:
- **OpenCV.js loads via a default import** (`import cvReady from '@techstark/opencv-js'`), NOT a namespace or
  dynamic `import()`. Its `module.exports` is a Promise, which a namespace/dynamic import turns into a broken
  thenable ("Promise.prototype.then called on incompatible receiver") in both Vitest and the browser build; a
  bundler default import returns `module.exports` (the real Promise) directly. In Vitest the engine CV tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaak0b/ScanNTune](https://github.com/jaak0b/ScanNTune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
