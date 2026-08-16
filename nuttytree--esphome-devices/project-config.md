---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is an [ESPHome](https://esphome.io) configuration repository: YAML device configs plus custom external components (Python codegen + C++) for a collection of home-automation devices (pool controller, HVAC zone control, energy monitors, smart plugs, etc.) that integrate with Home Assistant. It is not an application with a build/test pipeline in the traditional sense — the "build" is ESPHome compiling a device's YAML into firmware, and the "tests" are lint/format checks plus ESPHome's own YAML config validation.

Compiling and flashing devices normally happens through the ESPHome Dashboard/Device Builder, which mounts this repo's `devices/` directory as its config directory (its state files are described below). A local `esphome` CLI is also installed on this Windows machine via `pipx` (isolated from system Python) — useful for `esphome config devices/<file>.yaml` to validate a device's full resolved config without going through Device Builder; see the "ESPHome CLI" section below for details, gotchas, and update instructions. Full local compiles are possible too, but need one-time machine setup — see "Compiling locally".

## Commands

Linting/formatting is enforced via pre-commit (see `.pre-commit-config.yaml`) and run in CI on every push/PR (`.github/workflows/ci.yml`):

```
pre-commit run --all-files       # run every hook against the whole repo
pre-commit run --files <path>    # run against specific file(s)
```

Individual hooks, if you need to invoke a tool directly instead of via pre-commit:
- `ruff check --fix` / `ruff format` — lint + format for all Python (components).
- `flake8` — additional docstring/style checks, scoped to `components/**/*.py` only (`.flake8`).
- `yamllint` — all YAML files except `.clang-format`/`.clang-tidy` (`.yamllint`: 2-space indent, no line-length limit, `document-start` disabled).
- `clang-format` — C/C++ files in `components/**` (`.clang-format`).

### clang-tidy

`.clang-tidy` is a copy of upstream ESPHome's, so the C++ in `components/` is held to the same
checks as ESPHome core. It is **not** a pre-commit hook: clang-tidy needs a device's real
compiler flags and its generated `esphome/core/defines.h`, which only exist after a compile. CI
therefore runs it inside the compile matrix, right after each `esphome compile`.

`scripts/clang_tidy.py` is a slimmed-down port of upstream's `script/clang-tidy` (same flag
surgery — clang can't consume the xtensa/riscv GCC command line, so the flags are rebuilt from
the build's idedata). It takes the name of a *compiled* device config and checks only the
components that device built, with that device's defines:

```
pip install clang-tidy==22.1.8       # the version CI pins
python scripts/clang_tidy.py pool             # every component in that build
python scripts/clang_tidy.py pool pump_flow   # only files matching a regex
python scripts/clang_tidy.py pool --fix       # apply fix-its (runs serially)
```

A component that runs on both platforms needs a run per platform, same as the compile itself.
Locally that means compiling a scratch copy of the device first (see "Compiling locally") and
passing that copy's name.

Review what `--fix` writes before keeping it: it only sees the preprocessor branch the current
build compiles, so a rename inside `#ifdef USE_SENSOR` lands while the matching `#else`
definition keeps the old name — which builds fine here and breaks a device without that
feature. It also re-wraps what it touches in Google style; run clang-format afterwards.

### ESPHome CLI

Installed via `pipx install esphome`, which creates its own isolated virtualenv rather than touching system Python packages.

- **Must be installed under Python 3.12+**, not this machine's default Python 3.11 (`C:\Program Files\Python311`). ESPHome 2026.7.0+ requires Python `>=3.12,<3.15`; installing under 3.11 doesn't error, it silently resolves to the newest 3.11-compatible release instead (e.g. 2026.6.5 when 2026.7.2 was current), which can surface as spurious "Platform not found" errors for components added in newer releases. Use the `py -3.12` launcher (or the full interpreter path) when installing/upgrading:
  ```
  pipx install --python "C:\Users\<user>\AppData\Local\Programs\Python\Python312\python.exe" esphome
  ```
- The `esphome.exe` shim lands in the pipx bin dir (`%USERPROFILE%\.local\bin`), which `pipx ensurepath` adds to the persistent user PATH — new terminals/sessions should resolve `esphome` directly. A shell that was already running before the PATH update won't see it until restarted; fall back to the full venv path (`...\pipx\venvs\esphome\Scripts\esphome.exe` — check `pipx list` for the exact location, it can be nested as `pipx\pipx\venvs\...` depending on how pipx itself was installed) if `esphome` isn't found.
- To check/upgrade: `pipx upgrade esphome`, or reinstall with the `--python` flag above if it's drifted onto the wrong interpreter.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nuttytree/ESPHome-Devices](https://github.com/nuttytree/ESPHome-Devices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
