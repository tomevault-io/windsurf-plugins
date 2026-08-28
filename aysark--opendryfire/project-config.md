---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
uv sync                                   # install (Python 3.12+, uv required)
uv run pytest src/tests -q                # full suite, 154 tests, no hardware needed
uv run pytest src/tests/test_detect.py -q                        # one file
uv run pytest src/tests/test_detect.py::test_finds_a_dot_and_locates_it -q   # one test
uv run pytest src/tests -q -k "chime or click"                   # by name
uv run dryfire --help                     # CLI smoke checks, same as CI
uv run dryfire plan
uv run dryfire targets
```

`pyproject.toml` sets `pythonpath = ["src"]` and `testpaths`, so bare `uv run pytest` works too.
There is no linter or formatter configured — don't invent a `ruff`/`black` step.

The importable package is `opendryfire`; the installed command is `dryfire`
(entry point `opendryfire.cli:main`). CI (`.github/workflows/tests.yml`) runs the
suite on macOS and Ubuntu.

## The central idea

**The microphone answers *when*; the camera answers *where*.** Every design
decision downstream follows from that split, and violating it is the most
common way to break this codebase subtly:

- Timing is `(click_sample - beep_sample) / 48000`. The start beep is played and
  recorded on the *same* full-duplex stream as the striker click, so input and
  output latency cancel exactly. **Video timestamps are never used for timing** —
  only for pairing within a ±150 ms window (`session.PAIR_WINDOW_S`).
- Scoring is a homography from image pixels to target-plane inches. Camera
  pipeline delay is irrelevant to it.
- A shot requires **both** signals. Flash + click = scored shot; click alone =
  slide rack or off-target miss; flash alone = rejected. That fusion is what
  makes detection tractable in a red-lit room, and why thresholds only need to
  be good rather than perfect.

## Module map

| Module | Owns |
|---|---|
| `detect.py` | Blob finding + shot assembly. `LaserDetector.push(frame, t)` returns a `Shot` on the frame where a pulse *ends*. `calibrate_scene` derives room thresholds. |
| `audio.py` | `ShotTimer` (beep-and-listen, one rep), `LiveListener` (continuous, untimed drills), `ClickDetector`, `find_beep_onset`, `calibrate_click_floor`. |
| `geometry.py` | `TargetFrame` — the pixels→inches homography, its persistence, its scoring mask, and tripod-drift signatures. |
| `targets.py` | Target specs in **full-scale inches**, `ScaledTarget` (scale + stand-off → simulated distance), `group_stats`, shape classification. |
| `session.py` | `Shot`/`Rep`/`Session` records, `pair_events` fusion, JSON persistence, `~/.dryfire` path helpers. |
| `plans.py` | TOML plan loading and strict validation. |
| `drills.py` | Rules that are the same whoever wrote the plan: `ParLadder`, `evaluate_rep`, `coaching_note`. |
| `feedback.py` | Outcome chimes, constrained so the click detector cannot hear them. |
| `devices.py` / `camera.py` | Camera discovery by *name*, Continuity Camera preflight, capture that reports what the driver actually granted. |
| `ui.py` | The calibration click window (with magnifier) and the live drill overlay. |
| `cli.py` | Every subcommand; the only place modules are wired together. |
| `vault.py` | Optional Markdown/Obsidian export, off by default. |

## Coordinate and scale chain

Three spaces, and applying scale twice is the bug to watch for:

1. **Image pixels** → `TargetFrame.to_target()` →
2. **Printed-target inches** — origin at the A-zone centre, +x right, **+y up**
   (the homography flips image y). This is what `ShotRecord.x/y` stores.
3. **Full-scale inches** — `ScaledTarget.score()` divides by scale internally,
   so pass it printed inches. `to_sim_inches()` converts a *distance* to the
   simulated range.

Zone geometry in `targets.py` is written once at full scale for this reason.

## Runtime state — `~/.dryfire/`

`config.json` (camera + vault prefs) · `detector.json` (from `autocal`) ·
`audio.json` (from `audiocal`, and its existence is the "is timing calibrated?"
flag) · `calibration/<profile>.json` (the `TargetFrame`) · `sessions/*.json` ·
`plan.toml` (user plan, overrides the bundled one) · `spike/`.

Workflow order is load-bearing and documented that way everywhere:
**calibrate → autocal → spike → audiocal → run**. `autocal` is far better
informed once a calibration exists, because it can mask to the target plane.

## Two run paths

`cli.cmd_run` branches into `_run_timed` and `_run_untimed`, and they differ
more than they look. Timed mode opens a fresh full-duplex recording per rep and
has no live listener, so the guard against a chime being heard as a click is a
**wait** for the chime to finish; untimed mode runs `LiveListener` plus an
OpenCV window, and the guard is `listener.mute_for(...)`. Anything touching rep
flow, sound, or fusion usually has to change in both.

## Invariants that are easy to break

- **Reference points must be coplanar with the scoring surface.** The default is
  the target's own A-zone corners. Markers on a backer board are off by ~0.3″ —
  a quarter of a 1/5-scale A-zone, invisible in the output. ArUco is only for
  genuinely flat setups.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aysark/opendryfire](https://github.com/aysark/opendryfire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
