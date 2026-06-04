---
trigger: always_on
description: EEGPREP is a Python port of core EEGLAB preprocessing concepts, workflows, file names, data structures, GUI patterns, and user experience. Build features so EEGLAB users can predict where code lives and how APIs behave, while still using simple idiomatic Python when MATLAB style would make the code worse.
---

# Agent Guidelines for EEGPREP

EEGPREP is a Python port of core EEGLAB preprocessing concepts, workflows, file names, data structures, GUI patterns, and user experience. Build features so EEGLAB users can predict where code lives and how APIs behave, while still using simple idiomatic Python when MATLAB style would make the code worse.

Primary references:
- EEGLAB source: https://github.com/sccn/eeglab
- EEGLAB data structures: https://eeglab.org/tutorials/ConceptsGuide/Data_Structures.html

## Repo Map

- `src/eegprep/functions/popfunc/`: EEGLAB-style `pop_*` user-facing wrappers and `eeg_*` functions that operate on EEG structures, such as ICA wrappers. Keep each pop function in a `pop_<name>.py` module that mirrors `functions/popfunc/` in EEGLAB.
- `src/eegprep/functions/guifunc/`: EEGLAB-style GUI helpers such as `inputgui`, dialog specs, and Qt rendering. Keep GUI infrastructure parallel to `functions/guifunc/` in EEGLAB.
- `src/eegprep/functions/adminfunc/`: EEGLAB-style administrative helpers such as `eeg_checkset.py`, `eeg_options.py`, the main GUI launcher, and the synchronized `eegprep-console` workspace.
- `src/eegprep/functions/sigprocfunc/`: EEGLAB-style low-level signal processing functions such as `runica.py`, `runamica.py`, `topoplot.py`, `epoch.py`, and `eegrej.py`.
- `src/eegprep/plugins/clean_rawdata/`: Python ports of the EEGLAB clean_rawdata plugin, including `clean_*` and ASR modules.
- `src/eegprep/plugins/clean_rawdata/private/`: ports of clean_rawdata private helpers such as `fit_eeg_distribution`, `geometric_median`, FIR helpers, covariance helpers, and spherical-spline interpolation.
- `src/eegprep/plugins/ICLabel/`: Python ports of the EEGLAB ICLabel plugin and bundled `netICL.mat`.
- `src/eegprep/plugins/firfilt/`: Python ports of the EEGLAB firfilt plugin helpers.
- `src/eegprep/functions/miscfunc/`: EEGLAB-style miscellaneous helpers, including format conversion and numerical utilities.
- `src/eegprep/functions/eegobj/`: Python counterpart to EEGLAB's `functions/@eegobj/`.
- `src/eegprep/plugins/EEG_BIDS/`: Python ports and workflow helpers for the EEGLAB EEG-BIDS plugin.
- `src/eegprep/utils/`: Python-only test/development support. Do not put EEGLAB-equivalent processing code here.
- `src/eegprep/resources/`: MATLAB option files, montages, help text, and package data.
- `src/eegprep/resources/help/`: EEGPrep-owned Markdown help resources for GUI Help buttons and EEGLAB-style `pophelp` text. Add `<function_name>.md` here whenever new user-facing functionality needs GUI help.
- `src/eegprep/eeglab/`: vendored EEGLAB reference code for local development and parity work. Treat as reference input; do not edit unless explicitly updating the bundled reference. Do not make package runtime behavior depend on this directory existing.
- `tests/matlab/`: MATLAB parity scripts and MATLAB helper fixtures used by Python tests.
- `scripts/*.m`: MATLAB/Octave helper scripts that are not part of the normal unit-test tree.
- `sample_data/`: small checked-in EEG sample datasets, named to match EEGLAB's `sample_data` convention.
- `sample_notebooks/`: exploratory/sample notebooks. Keep runnable examples in docs when they are user-facing.
- `tools/`: developer and parity tooling that is not installed as part of the `eegprep` package.
- `tests/`: `unittest` tests. Test files generally mirror source module names.
- `docs/source/`: Sphinx docs, examples, API pages.
- `.github/workflows/test.yml`: CI test and pre-commit entry points.
- `pre-commit.py`: required lint/check script for this repo.

## Before Coding

- Check whether a matching skill exists. Skills are task-focused playbooks in `.agents/skills/` and are also accessible as `.claude/skills/`. Before starting any non-trivial task, scan the skill descriptions in your system prompt; if one matches, invoke it via the Skill tool instead of using ad-hoc commands.
- Use `.agents/skills/eeglab-gui-visual-parity/SKILL.md` when building or iterating on EEGPrep GUI features, especially `pop_*` dialogs that should match EEGLAB screenshots through the visual parity capture loop.
- State assumptions before implementing. If the request has multiple plausible interpretations, present them.
- If something is unclear, stop and ask. Do not hide confusion in code.
- If a simpler approach exists, say so. Push back on speculative features, compatibility shims, or unnecessary abstractions.
- For multi-step work, state a short plan with verification for each step. Include code snippets when they clarify the intended change.
- Define verifiable success criteria. For example: bug fix means reproduce with a failing test, implement, then pass the test; feature means update behavior, tests, docs, and pre-commit.

## EEGLAB Parity

- Use EEGLAB as a development and parity oracle, not as an EEGPrep runtime dependency. During development, compare against EEGLAB so EEGPrep features look, feel, and behave like EEGLAB for EEGLAB users; at runtime, EEGPrep must work standalone without an EEGLAB checkout.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sccn/eegprep](https://github.com/sccn/eegprep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
