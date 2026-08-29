---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Windows/CUDA VFX matchmove batch tool: raw plate frames in, 2D point tracks for
3D Equalizer out. Four stages chained behind one local NiceGUI app:

1. **Analyze** — Qwen2.5-VL describes each shot (movers, occluders, bad-track regions,
   depth layers, parallax) → deterministic heuristics turn that into SAM3 include/exclude
   prompts and a per-shot track strategy.
2. **Mask** — SAM3 renders per-frame keep/ignore alpha mattes from those prompts.
3. **Track** — SynthEyes (default) or TAPNext++ (GPU fallback), mask-gated.
4. **Export/publish** — classic 3DE 2D-track ASCII, copied into the studio shot tree.

Everything runs offline and locally: weights are loaded with `local_files_only=True` and
will error rather than download.

## Commands

```bat
app\BotBatchTracker.bat                      REM the launcher: NiceGUI app on :8080
```

The bat sets `TMP`/`PIP_CACHE_DIR` inside `runtime/`, exports `BTR_SAM3_WEIGHTS` and the
SynthEyes paths, kills whatever holds port 8080, then runs `app/app_nicegui.py` with
`runtime\python311\python.exe` (or `.venv` if present). **cwd must be the repo root** —
`app/` is both a package and the dir holding `app.py`.

There is no test framework. Verification is done with three scripts, all pure-Python
(no torch/cv2 except `check_per_track`), run from the repo root:

```bat
runtime\python311\python.exe tools\check_per_track.py
REM self-check of the per-track-policy plumbing on synthetic frames (~seconds, no GPU).
REM Catches seed-measurement/track-column misalignment, which otherwise produces a
REM successful run with quietly wrong numbers.

runtime\python311\python.exe tools\eval_refs.py refs\gt4k --bot <export>.txt
runtime\python311\python.exe tools\eval_refs.py refs\gt4k --bot new.txt --baseline refs\gt4k\baseline_bot.txt
REM one row per hand-tracked reference, labelled by feature kind (corner/blob/edge/dense).
REM This is the gate for any tracking-accuracy change: an averaged single-track number
REM hides a rule that helps corners and hurts blobs.
REM NOTE: refs\gt4k holds ONE corner, and the plate it came from is not on this machine --
REM its locked numbers (7.00/4.18/0.94/1.22/1.36) can be re-read but not re-run. Producing a
REM NEW measurement needs bench\ below, or a reference whose footage is present.

runtime\python311\python.exe bench\make_synth.py --out bench\synth\lab02 --frames 100 --hard
runtime\python311\python.exe bench\run_bench.py   --shot bench\synth\lab02 --tag base
runtime\python311\python.exe bench\score_synth.py bench\synth\lab02 --run test --baseline base
REM synthetic shot with EXACT ground truth for every seed, scored per feature class. Covers
REM what one hand-tracked corner cannot; measures localisation only (one plane -> no parallax,
REM no occlusion, and every track on it comes out good). See bench\README.md.

runtime\python311\python.exe -m app.compare_tracks bot.txt manual.txt --bot-track BWD_0010 --ref-track 658
REM one bot track vs one artist track: deviation, first-step error, fast-vs-slow bias,
REM smoothness, worst frames. Use while chasing a single fault.
```

Reference folder layout (`refs/<name>/`): `manual.txt` (hand tracks, 3DE ASCII),
`refs.json` (`{"658": "corner", ...}`), optional `baseline_bot.txt`, `baseline.json`.
Pairing is positional-by-proximity, not by name.

```bat
runtime\python311\python.exe tools\make_lk_reference.py --mp4 D:\Jefrin\IN\SH004.mp4 ^
    --out refs\SH004_lk --quality 0.01 --min-dist 30 --max-closure 1.0 --n 6
REM Builds a reference for a real plate when no hand track exists, using pyramidal
REM Lucas-Kanade -- a different algorithm family from the bot's NCC+ECC refine, so the two
REM do not share pixel-locking bias. Each track is run forward then all the way back; the
REM round-trip CLOSURE bounds that reference's own precision and is recorded per track in
REM reference.json. NOT ground truth: a fault both methods share is invisible to it, so read
REM the closure before believing any row. Folders are named `_lk` to keep the provenance
REM obvious next to real hand tracks.
```

Model weights, the portable interpreter, `out/`, `_batches/`, and `backup/` are
gitignored. `CHECKPOINTS.txt` is the by-hand weight download sheet; `setup_bot.bat`
does it automatically.

## Architecture

### UI / backend split
`app/app.py` (~3.2k lines) is the **entire backend** — loaders, workers, `AppState`,
path helpers — plus a legacy Gradio UI that only builds under `__main__`.
`app/app_nicegui.py` is the live front-end: it loads `app.py` as a file under the module
name **`btr_backend`** (aliased `be`) because the sibling `app/` package would shadow
`import app`. Adding backend behaviour means editing `app.py` and reaching it through
`be.` from the UI.

### Module loading (deliberate, fragile, don't "clean up")
- The embeddable interpreter's `._pth` blocks auto-path insertion, so `_bootstrap_paths()`
  in `app.py` rglobs for `core/io_parsers.py` and injects its parent. The decisioning
  package is imported as `core.*` but **lives at `pipeline/llama/core/`**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antonyjaguar19-dotcom/Batch_tracker_repo](https://github.com/antonyjaguar19-dotcom/Batch_tracker_repo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
