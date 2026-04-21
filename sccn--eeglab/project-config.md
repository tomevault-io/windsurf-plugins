---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is EEGLAB?

EEGLAB is an open-source MATLAB toolbox for processing electrophysiological signals (EEG, MEG, and other time series data). It provides a GUI and command-line functions for continuous and event-related data analysis, including independent component analysis (ICA).

## Running MATLAB

```bash
# Non-interactive batch mode (preferred for automation)
/Applications/MATLAB_R2025a.app/bin/matlab -batch "command_here"

# Example: run EEGLAB without GUI and process data
/Applications/MATLAB_R2025a.app/bin/matlab -batch "cd('/Users/arno/GitHub/core_eeg/eeglab'); eeglab nogui; your_script"
```

Startup options: `eeglab` (full GUI), `eeglab nogui` (headless), `eeglab redraw` (refresh GUI), `eeglab rebuild` (close and rebuild).

## Git Workflow

- **`develop`** - Main and default branch
- Submodules: `dipfit`, `clean_rawdata`, `ICLabel`, `firfilt`, `EEG-BIDS`, `tutorial_scripts`
- Clone with `--recurse-submodules`; update with `git submodule update --init --recursive --remote`

## Code Architecture

### The EEG Structure

All processing revolves around the `EEG` struct:

| Field | Type | Description |
|-------|------|-------------|
| `data` | `[chan x pts]` or `[chan x pts x epochs]` | Raw data matrix |
| `nbchan`, `pnts`, `trials` | int | Dimensions (trials=1 for continuous) |
| `srate` | float | Sampling rate in Hz |
| `xmin`, `xmax` | float | Epoch time bounds in seconds |
| `times` | vector | Latency vector in milliseconds |
| `chanlocs` | struct array | Channel names/locations |
| `event` | struct array | Events with `.type`, `.latency`, `.duration` |
| `urevent` | struct array | Original events before any rejection |
| `epoch` | struct array | Epoch metadata (only when epoched) |
| `ref` | string/int | Reference type (`'common'`, `'averef'`, channel index) |
| `icaweights` | matrix | ICA unmixing weights |
| `icasphere` | matrix | ICA sphering matrix |
| `icawinv` | matrix | ICA inverse (mixing) matrix |
| `icaact` | matrix | Component activations (may be empty; recomputed on demand) |
| `dipfit` | struct | Dipole model for ICA components |
| `reject` | struct | Rejection marks (`.gcompreject` = flagged components) |
| `etc` | struct | Miscellaneous (ICLabel results stored here) |
| `history` | cell | Command history for reproducibility |

Always call `eeg_checkset(EEG)` after modifying the structure to validate and recompute derived fields.

### Three Function Categories

1. **`pop_*` functions** (`functions/popfunc/`): GUI wrappers that show dialogs, call processing functions, return `[EEG, LASTCOM]`. Entry points from menus.
2. **`eeg_*` functions** (`functions/adminfunc/`, `functions/popfunc/`): Structure manipulation and validation (`eeg_checkset`, `eeg_epoch`, `eeg_store`).
3. **Processing functions** (`functions/sigprocfunc/`, `functions/timefreqfunc/`): Direct signal processing (`runica`, `topoplot`, `spectopo`, `eegfilt`).

### Plugin Architecture

Plugins live in `plugins/` and register via `eegplugin_[name].m`.

Browse all available plugins: https://sccn.ucsd.edu/eeglab/plugin_uploader/plugin_list_all.php

Install plugins programmatically:
```matlab
% plugin_askinstall(plugin_name, plugin_function, interactive)
% interactive: 0 = install silently, 1 = prompt user
plugin_askinstall('ICLabel', 'iclabel', 0);        % install ICLabel
plugin_askinstall('clean_rawdata', 'clean_artifacts', 0);
plugin_askinstall('firfilt', 'pop_eegfiltnew', 0);
plugin_askinstall('picard', 'picard', 0);
plugin_askinstall('dipfit', 'pop_dipfit_settings', 0);
```

## EEGLAB Menu-to-Function Reference

### File Menu
| Menu Item | Function |
|-----------|----------|
| Load existing dataset | `pop_loadset` |
| Save current dataset(s) | `pop_saveset` |
| Import data from file | `pop_fileio`, `pop_biosig`, `pop_importdata` |
| Import events | `pop_importevent` |
| Import epoch info | `pop_importepoch` |
| Export data to text | `pop_export` |
| Preferences | `pop_editoptions` |
| Import BIDS dataset | `pop_importbids` (EEG-BIDS plugin) |

### Edit Menu
| Menu Item | Function |
|-----------|----------|
| Dataset info | `pop_editset` |
| Channel locations | `pop_chanedit` |
| Event fields | `pop_editeventfield` |
| Event values | `pop_editeventvals` |
| Select data (channels/time) | `pop_select` |
| Select data using events | `pop_rmdat` |
| Select epochs or events | `pop_selectevent` |
| Append datasets | `pop_mergeset` |

### Tools Menu (Preprocessing)
| Menu Item | Function |
|-----------|----------|
| Change sampling rate | `pop_resample` |
| Basic FIR filter (legacy) | `pop_eegfilt` |
| FIR filter (firfilt plugin) | `pop_eegfiltnew` |
| Re-reference | `pop_reref` |
| Interpolate electrodes | `pop_interp` |
| Inspect/reject by eye | `pop_eegplot` |
| Automatic channel rejection | `pop_rejchan` |
| Automatic continuous rejection | `pop_rejcont` |
| Automatic epoch rejection | `pop_autorej` |
| Decompose data by ICA | `pop_runica` |
| Remove components from data | `pop_subcomp` |
| Extract epochs | `pop_epoch` |
| Remove epoch baseline | `pop_rmbase` |
| Clean Rawdata and ASR | `pop_clean_rawdata` (plugin) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sccn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
