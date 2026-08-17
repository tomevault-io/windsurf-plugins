---
trigger: always_on
description: generates the next one.
---

# AGENTS.md

This file provides guidance to agents when working with code in this repository.

It is a map, not a manual: each entry says what a module is for and what would
break if it were changed carelessly. The reasoning behind a specific line lives
in the comment next to that line.

## Project Overview

Mimora is a local, offline **pronunciation trainer** (Python 3.11/3.12, Tkinter
GUI). It speaks an LLM-generated phrase aloud, records the user repeating it,
then scores the attempt against the reference with the active pronunciation
engine plus prosody. The user repeats the same phrase until satisfied, then
generates the next one.

The engine is selected by `config.ENGINE` (settings.json `"engine"`, dispatched
by [`mimora/engine.py`](mimora/engine.py)):

- **`phoneme`** (default) - espeak reference phonemes vs a wav2vec2 phoneme
  recognizer, feature-weighted edit distance, calibrated 0-5 grade.
- `acoustic` - Wav2Vec2-embedding cosine-DTW plus phoneme/word error rates.
  English-only.
- `none` - scoring off. No recognizer is loaded, every take is accepted, the
  GUI shows a neutral read-out. For slow machines.

The **practice language is data, not an assumption**: every language is one
entry in `config.LANGUAGE_PROFILES`, assembled from the pure-data modules in
[`mimora/languages/`](mimora/languages/) (the profile format is documented in
that package's `__init__.py`). The active language/variant comes from
settings.json `"practice_language"` / `"accent"` and applies after a restart.
English (`american`/`british`) is fully calibrated; Spanish (`castilian`) runs
the phoneme engine as **experimental** until
`pronunciation/phoneme/es_model_calibration.json` lands. Adding a language is a
profile module plus an engine calibration, never a new `if language == ...`.

The pronunciation-scoring core in `pronunciation/acoustic/` is adapted from
[OpenPronounce](https://github.com/Halleck45/OpenPronounce) (MIT) and reused as
a GUI-agnostic library.

## Running the App

```bash
pip install -e .
python main.py
```

Editable on purpose: a plain `pip install .` from a clone leaves a second copy
in site-packages that your edits never reach, and switches `paths.py` out of
source-tree mode. Dependencies come from `[project.dependencies]` and cover both
engines; the two `requirements.txt` under `pronunciation/` remain because those
subpackages are installable on their own.

Three launch forms, differing in nothing after `mimora/cli.py`'s `main()`: the
root `main.py` shim for a checkout, `python -m mimora`, and the `mimora` console
script.

Also requires a GGUF chat model at `config.EXTERNAL_MODEL_PATH`. **espeak-ng is
not a separate install**: the pinned `espeakng-loader` wheel ships the library
and [`pronunciation/common/espeak.py`](pronunciation/common/espeak.py) registers
it with `phonemizer`. On Linux add **PortAudio** (`libportaudio2`) - the
`sounddevice` wheel bundles it on Windows and macOS but not there.

**Default LLM backend**: `llama-server`, the official llama.cpp binary launched
as a subprocess ([`mimora/llm_server_ctl.py`](mimora/llm_server_ctl.py)).
`install.py` installs it (`step_llama_server`); the app fetches whatever is
missing on first run, asking first. **Alternatives**: `"llm_backend":
"lm-studio"` (LM Studio on `http://localhost:1234`, or `"lm_studio_host"`
elsewhere on the network), or `"off"` - no LLM at all, phrases are the source
text's own sentences taken verbatim in order
([`mimora/phrase_source.py`](mimora/phrase_source.py)). Nothing in the project
installs or imports llama-cpp-python.

## Architecture

### Controller and view

- [`mimora/app.py`](mimora/app.py) - `PronunciationTrainerGUI`: the Tkinter
  controller, recording, the Prompt→Record→Analyze→Feedback→Loop state machine,
  threading orchestration. Module-level `run()` is the startup sequence (root
  logging, first-run window, GUI) and the only thing `cli.py` calls.
- [`mimora/cli.py`](mimora/cli.py) - the console-script entry point. Exists
  purely for ordering: `bootstrap.early_init()` only works before the libraries
  it configures are imported, and `--version` must answer without waiting for
  torch, so neither can live in `app.py`. The function-local `from mimora import
  app` is load-bearing, not a style choice. **Stdlib-only at module level.**
  [`mimora/__main__.py`](mimora/__main__.py) is a shim over the same `main()`.
- [`mimora/bootstrap.py`](mimora/bootstrap.py) - early process setup,
  stdlib-only. Two phases that **must not be merged or reordered**:
  `early_init()` runs before the heavy imports (from `cli.py`),
  `setup_logging()` after them (from `app.run()`, `force=True` replacing
  handlers installed during the imports). Owns log continuity across an
  in-session restart (`--append-log` / `APPEND_LOG_FLAG`) and the
  `DISABLE_SAFETENSORS_CONVERSION` switch.
- [`mimora/ui.py`](mimora/ui.py) - `TrainerView`: the view facade `app.py`
  composes. Window chrome, control row, `enter_*` intent methods, feedback
  orchestration; delegates panel-local work to
  [`ui_practice.py`](mimora/ui_practice.py) (source-text editor),
  [`ui_hero.py`](mimora/ui_hero.py) (phrase, translation, score row),
  [`ui_prosody.py`](mimora/ui_prosody.py) (sparklines) and
  [`ui_history.py`](mimora/ui_history.py) (attempt list). Shared palette, fonts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vikonix/Mimora](https://github.com/vikonix/Mimora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
