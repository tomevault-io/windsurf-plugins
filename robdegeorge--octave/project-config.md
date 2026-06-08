---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OCTAVE (Open-source Cross-platform Telematics for Augmented Vehicle Experience) is a **C++ / Qt 6 / QML** infotainment system designed for vehicles. It runs on Windows, macOS, Linux, Raspberry Pi, iOS, and Android.

### Backend state: C++ is the only shipped binary; Python is a developer/tinkerer backend

OCTAVE intentionally maintains **two parallel backends** — C++ / Qt 6 (`src/`) and Python / PySide6 (`backend/`, `main.py`) — that expose the same API surface to the shared QML frontend (`frontend/`). This is deliberate: Python stays alive for tinkering, accessibility, rapid prototyping, and hardware hacking on Raspberry Pi / single-board setups where a Python REPL is invaluable; C++ is the **sole distribution path** — every shipped binary on every platform (Windows `.exe`, macOS `.dmg`, Linux AppImage / `.deb`, Android `.apk`, iOS `.ipa` once enabled) is built from the C++ tree. Python is not compiled or packaged by CI; tinkerers run it via `python setup.py` or `python main.py` from a checkout.

**Mobile (Android/iOS) is C++-only.** The Python backend is desktop-only (Linux / Windows / macOS / Raspberry Pi). Python-on-Android via buildozer/p4a has been removed — do not reintroduce `backend/platform_config.py`, `backend/stubs.py`, `backend/android_obd_manager.py`, `backend/android_sensors.py`, `deployment/buildozer.spec`, or `requirements-android.txt`. Mobile-specific code lives behind `#ifdef Q_OS_ANDROID` / `#ifdef Q_OS_IOS` in C++ and does **not** need a Python mirror.

**Parity is the rule, not the goal — on desktop.** When you add a feature, fix a bug, or change behavior in one backend, you must do the equivalent work in the other in the same change set for desktop functionality (or explicitly note why parity is deferred and open a `TODO/` entry). The QML frontend must continue to work identically against either backend — no QML file should know or care which backend is running, beyond `typeof someManager !== "undefined"` guards for features legitimately absent on one platform.

**Concrete parity expectations (desktop):**
- Every C++ manager in `src/managers/foo.{h,cpp}` has a Python peer at `backend/foo.py` (and vice versa) for desktop functionality, with the same class name, same QML context property name, same public Slots/Signals/Properties, and equivalent semantics.
- Settings schemas must match: a key introduced in `SettingsManager` (either language) is added to the other in the same commit.
- JSON persistence formats (e.g. settings, dashboards) are shared on disk — both backends must read and write compatible files.
- Volume curves, threading models, logging categories/loggers, image providers, and custom QML types are mirrored.
- When a QML file gains a new binding or call (`fooManager.bar()`), both backends must expose `bar` before the QML ships — *unless* the binding is mobile-only, in which case only the C++ side needs `bar`.

**Historical migration plan:** `.claude/plans/wondrous-toasting-biscuit.md` describes the original Python→C++ rewrite. It is **superseded** by this dual-backend policy — Python is no longer slated for deletion.

## Commands

### C++ build (primary)

```bash
# Configure + build (debug)
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j

# Run
./build/octave

# App store build (downloads feature stripped out)
cmake -S . -B build -DOCTAVE_ENABLE_DOWNLOADS=OFF
cmake --build build -j
```

See `BUILD.md` for the full build matrix (9 targets including iOS / Android / Flatpak / app store variants).

### Python build (equal, supported)

```bash
# First-time setup + run (creates venv, installs deps, launches app)
python setup.py

# Setup only, don't launch
python setup.py --no-run

# Run the app directly (activate venv first)
source venv/bin/activate
python main.py

# Debug mode (enables debug logging)
python main.py --debug

# Developer mode (simulated OBD data, keyboard controls for testing)
python -m dev.main_dev

# Lint (ruff — configured in pyproject.toml)
source venv/bin/activate
ruff check .

# Smoke tests (headless — imports + safe manager instantiation)
QT_QPA_PLATFORM=offscreen pytest tests/
```

A minimal smoke test suite lives in `tests/` and runs in CI on every push and PR to `main`. Ruff runs in warn-only mode during Phase 1 cleanup — see the TODO in `.github/workflows/build.yml` for when to flip critical rules (E722, F821) to blocking.

## Architecture

**Two parallel backends (C++ / Qt 6 in `src/` and Python / PySide6 in `backend/`) + a shared QML frontend** (`frontend/`) communicating via Qt Signals/Slots. Both backends expose the same manager API surface to QML; any work in one must be mirrored in the other.

### Backend — C++ (`src/`, primary for builds & distribution)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RobDeGeorge/OCTAVE](https://github.com/RobDeGeorge/OCTAVE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
