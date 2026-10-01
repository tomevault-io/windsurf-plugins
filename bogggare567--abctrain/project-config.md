---
trigger: always_on
description: One app and three plugins in one CMake build (ADR 041): **abcTrain** (the
---

# abcTrain — notes for a coding session

One app and three plugins in one CMake build (ADR 041): **abcTrain** (the
`EarTrainer` target, standalone only — nine listening exercises on a
3-down/1-up staircase, plus a Studio tab that runs the Learners inside the
app) and **ABC Learner EQ / Comp / Verb** (VST3/AU — real processors that
teach while you use them). Vendor `soundkorb`. Product names begin with `ABC`; CMake
targets and folders keep the short forms (`EarTrainer`, `LearnerEQ/`).
Renaming a product changes its VST3/AU id and breaks saved host projects —
don't.

The person who owns this is a practising sound engineer. **Answer in
Russian.** He values direct criticism and explanation through
understanding.

## Where things are

| | |
|---|---|
| [docs/orientation.md](docs/orientation.md) | the map, the load-bearing ideas, **the rules from the literature** — read first |
| [docs/code-map.md](docs/code-map.md) | per-file breakdown (was the body of this file) — read the part you change |
| [docs/decisions/](docs/decisions/) | ADRs, 001–047: why each shape was chosen |
| [docs/process.md](docs/process.md) | how a task goes from idea to release; the scenario checklist |
| [docs/research/](docs/research/) | the literature review every current proposal rests on |
| [docs/design/](docs/design/) | sound library, education/live, the redesign spec |
| [docs/roadmap.md](docs/roadmap.md) | direction in broad strokes |
| public board | GitHub Project «abcTrain»; `tools/board/cards.json` + `create_board.py` |
| [docs/wiki/](docs/wiki/) | the user manual (EN + RU), mirrored to the GitHub wiki |

Concrete tasks live on the board, not in this file.

## Build and check

```
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
ninja -C build EarTrainerTests && ./build/EarTrainerTests_artefacts/Release/EarTrainerTests
ninja -C build ClickMap EditorSnapshots
xvfb-run -a ./build/ClickMap_artefacts/Release/ClickMap
SNAP_DARK=1 SNAP_SIZE=1100x780 xvfb-run -a ./build/EditorSnapshots_artefacts/Release/EditorSnapshots <dir>
```

- Build targets separately in the cloud container (two cores, LTO): a full
  `ninja` does not fit in ten minutes. Configure once, then incremental.
- `EditorSnapshots` renders every screen to PNG — the only way to see a
  layout from the container. **Look at the picture before saying a visual
  change works** (skill `juce-editor-render-check`).
- `ClickMap` checks that every control a screen shows can be clicked, that
  every Studio plugin opens again after switching, and that a page from the
  top bar silences the exercise.
- **Text:** never `g.drawText` / `g.drawFittedText` directly — use
  `AbcTrainLookAndFeel::fitText` / `fitLines` (they shrink before they cut).
  `TEXT_AUDIT=1` on EditorSnapshots lists every line that still does not fit;
  run it on all 12 languages before calling a screen done.
- `tests/RealtimeSafetyTest` enforces the audio-thread rule below.
- `EarTrainerTests <name>` runs one test (or category). `GoldenAudio`
  compares each plugin's output with `tests/golden/*.flac`; a change that is
  *meant* to change the sound regenerates them (`GOLDEN_WRITE=1`) and says
  why in the commit. `REVERB_REPORT=1 … ReverbCharacter` prints what every
  reverb knob measurably does.
- Bots: `python3 tools/bots/train_bots.py` regenerates `Source/BotWeights.h`
  (never edit that by hand).

## Structure (ADR 039, 040)

```
shared/ui/        abc_ui        theme, look-and-feel, fonts, icons, widgets, window fit
shared/analysis/  abc_analysis  spectrum, waveform, meters, vectorscope     → ui
shared/audio/     abc_audio     library + packs, slicer, beds, generators   → (nothing)
shared/dsp/       abc_dsp       reverb, compressor, EQ coefficients (header-only)
shared/learning/  abc_learning  modules, lessons, Learner editor base, A/B  → ui analysis audio dsp updates
shared/updates/   abc_updates   version, update check                       → ui
shared/i18n/      abc_i18n      localisation (12 languages)
Source/Games + managers         abc_trainer_engine — the nine exercises and their rules, no GUI
Learner*/Source (no PluginEntry) abc_learner_eq/comp/verb, abc_learners — plugins and the app's Studio
```

- Includes from the repository root: `#include "shared/dsp/ReverbEngine.h"`.
- A new shared `.cpp` goes in its group's `target_sources` once. A new
  exercise goes in `abc_trainer_engine` **and is appended** to
  `GameManager`'s list (progress is keyed by index).
- Nothing in `shared/` includes `Source/` or `Learner*/`. `Source/` may
  include `Learner*/Source/` (the Studio) — never a `PluginEntry.cpp`.
- One engine per effect: the trainer and the plugins run the same DSP.
  Don't add a second reverb or compressor.

## Rules that are not up for silent change

- **Audio thread** (ADR 038): inside `processBlock` and anything it calls —
  no allocation, lock, file I/O, `String` building, `ValueTree` edit, GUI
  call or wait. Data leaves through atomics or lock-free FIFOs.
- **Answer mechanics:** no "Submit" button in the trainer; moving the scale
  never plays the answer (literature: ISA vs ICA — see orientation.md).
- **Answer names and technical parameter names are not translated**
  (Room, Plate, Low-mids; Threshold, Ratio, Attack, Pre-delay, Q, Bell,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bogggare567/abcTrain](https://github.com/bogggare567/abcTrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
