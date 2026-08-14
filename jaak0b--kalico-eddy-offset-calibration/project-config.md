---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A Kalico (Klipper fork) plugin that calibrates per-tool XYZ nozzle offsets on toolchanger 3D
printers using a bed-mounted LDC1612 eddy-current sensor board. Non-contact and immune to dirty
nozzles: the sensor sees only metal. XY comes from directional scans over the coil with parabolic
sub-sample fitting of the symmetric response (material-independent); Z comes from a
frequency-vs-height descent curve anchored by a one-time per-tool contact reference. Offsets are
printed to the console in v1; no toolchanger integration yet. Hardware is chengxg's open-source
"Little Crab" dual-coil board (self-assembled; see `project.md`). Design: `docs/design.md`;
session state: `HANDOVER.md`.

## The plugin: Python, Kalico klippy plugin

Single module `eddy_tool_calibration.py`, installed by symlink into Kalico's gitignored
`klippy/plugins/` directory (Kalico loads it like an extra; config section name = module name).
GPLv3 (algorithm ported from the GPLv3 upstream, kept in `reference/`). Kalico-only target; no
stock-Klipper compatibility requirement.

Commands (repo root):

```bash
pip install -r requirements_test.txt
python -m pytest tests/            # unit tests for fit math and scan geometry
```

Structure:

- **`eddy_tool_calibration.py`**: the plugin. Pure-math parts (parabolic fit, pair averaging,
  center reconstruction, curve evaluation) live in functions with no klippy dependency so they
  are unit-testable standalone.
- **`reference/tool_eddy_calibration.py`**: upstream file, unmodified, algorithm provenance.
- **`kalico/`**: shallow read-only clone of Kalico for API reference (not shipped, gitignored).
  Files we build against: `klippy/extras/ldc1612.py`, `probe_eddy_current.py`,
  `motion_report.py`, `tools_calibrate.py`.
- **`tests/`**: pytest; synthetic response curves with known centers as oracles.
- **`docs/`**, **`project.md`**, **`HANDOVER.md`**: design, hardware state, session handover.

Durable gotchas:
- Inside `klippy/plugins/`, cross-module imports must use the full namespace
  (`from klippy.extras import ldc1612`), never relative imports.
- Our crab board's LDC1612 CLKIN is 24 MHz (`frequency: 24000000`); the BTT Eddy Coil dev unit
  differs (driver default 12 MHz). Wrong constant scales all frequencies; geometry math is
  scale-free but Z references are not transferable across clock settings.
- `ldc1612.add_client` delivers 0.1 s batches of `(print_time, freq_hz, dummy_z)` at a fixed
  250 Hz; watch the `errors`/`overflows` fields at scan edges.
- Map sample timestamps to positions with `motion_report.get_trapq_position(print_time)`,
  never by assuming constant scan velocity.
- **The `kalico/` clone is newer than the printer this runs on.** Reading an attribute,
  option or method there proves it exists in current Kalico, not in the owner's build. Before
  depending on any Kalico surface, establish when it was added and reach it defensively if it
  postdates the target build (`ldc1612`'s `frequency` option and its `freq_conv` attribute both
  arrived 2026-03-04; the printer runs December 2025). The tests cannot catch this: they never
  import klippy, so every Kalico-facing line is proven only on hardware.
- An unexpected exception in a gcode handler is not an error message, it is a printer shutdown.
  Klipper's dispatcher only catches `CommandError`, so an `AttributeError` or a `KeyError`
  reaches the bare handler and shuts the machine down mid-command.

## Conventions

Numbered for unambiguous reference; do not cite rule numbers in shipped source or UI text.

1. **Measurement integrity: established methods only, never a fudge.** Every algorithm
   (parabolic sub-sample fitting, forward/reverse pair cancellation, least-squares center
   reconstruction, frequency-curve evaluation) is an established method or ported from the
   validated upstream, named with provenance in comments. NEVER introduce a hand-tuned constant,
   empirical offset, or bias correction fitted to make one particular setup's numbers look
   right: that overfits the sample and lies on the next one. Tunable tolerances are config
   parameters, exposed and documented, not buried magic numbers.

2. **No silently swallowed errors.** An `except` must surface the error, rethrow, or return a
   value the caller can act on. Every failure path a user can hit (no samples, too few samples,
   bad fit residual, no extremum in window, sensor amplitude errors) raises a gcode error with
   an actionable message naming the likely cause.

3. **Keep the math framework-agnostic and modular.** Fit and geometry functions take plain
   numbers/arrays and import nothing from klippy; the plugin class is the only klippy-facing
   layer. New calibration modes or sensor variants are their own modules behind clear
   interfaces.

4. **NO AI attribution in git/GitHub; no AI process residue in any output.** A
   `Co-Authored-By: Claude <...>` trailer NEVER allowed on commits. no AI
   attribution anywhere. Commit messages: a single short sentence. Shipped output of every kind
   (source code, comments, docstrings, docs, UI text, error messages, commit messages) must

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaak0b/kalico-eddy-offset-calibration](https://github.com/jaak0b/kalico-eddy-offset-calibration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
