---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## What this is

`cpm-fm` is a **PySide6 (Qt for Python)** desktop app that transfers files between a modern host and
legacy [CP/M](https://en.wikipedia.org/wiki/CP/M) systems over a serial link using X-Modem. Beyond
single/multi-file transfers it offers drag-and-drop, file-conflict resolution, CP/M 8.3 filename
validation, a persistent transfer history, whole-drive backup/restore, and a 12-language UI. The GUI
applies a Material Design theme via the `qt-material` package (set centrally at start-up in
`gui/theme.py`; the light/dark variant follows the host OS — UIR-070/UIR-073). As of v1.3 the UI was
migrated from Tkinter to PySide6; there is no remaining `tkinter` code. The project's Python
interpreter lives in **`.venv/`** — every `python`/`pip`/`pytest`/`mypy`/`ruff` command MUST run
through it (see Commands below; a PreToolUse hook blocks bare invocations). The stray `wxPython`
*package* inside `.venv` is unused (ignore it). The current version is in `src/version.txt`.

## Commands

> **MANDATORY: use the `.venv` interpreter for everything Python.** Run every `python`, `pip`,
> `pytest`, `mypy`, and `ruff` command as `.venv/Scripts/python.exe -m <tool> …` — never the bare
> command, which may resolve to the wrong interpreter. This repository rule applies regardless of
> whether the active development tool enforces it automatically. The commands below already use the
> required form; keep it when copying them, and invoke the `.venv` interpreter directly rather than
> hiding a bare tool behind a shell prefix.

Install (editable, with dev tools): `.venv/Scripts/python.exe -m pip install -e .[dev]`

Run:
- `cpm-fm` — installed GUI launcher (bound to `pythonw.exe`, no console window on Windows)
- `.venv/Scripts/python.exe -m cpm_fm` — equivalent, keeps a console for `print()` debug output
  (serial errors are printed to stdout, not surfaced in the UI)

Test / lint / type-check (CI runs all of these on Python 3.12, see `.github/workflows/ci.yml`):
- `.venv/Scripts/python.exe -m pytest` — full unit suite (`-q` is set in `pyproject.toml`;
  `testpaths = ["tests"]`, so it never descends into `integration/`)
- `.venv/Scripts/python.exe -m pytest tests/test_cpm_parser.py::test_parse_dir_output_extracts_filenames` — single test
- `.venv/Scripts/python.exe -m ruff check src tests` and `.venv/Scripts/python.exe -m ruff format --check src tests`
  (CI uses `--check`; drop it to apply)
- `.venv/Scripts/python.exe -m mypy src`

Integration (hardware-in-the-loop) suite — **separate, bench-only, not run by CI or the default
`pytest`** because it drives the real app against a real CP/M machine over serial:
- `.venv/Scripts/python.exe -m pytest integration/` — the HIL suite (its own `integration/pytest.ini`);
  `--target`/`--all-targets` select the rig, `--run-destructive` enables the backup/restore (whole-drive-wipe) cases
- `.venv/Scripts/python.exe integration/run.py` — interactive target picker
- See `integration/README.md` for wiring, target setup (`hil_config.json`), and the manual-vs-automated
  split. It is additive test infrastructure and does not itself define requirements.

## Architecture

**Authoritative reference: [`docs/cpm_fm_architecture.md`](docs/cpm_fm_architecture.md)** — the
Software Architecture Description. It holds the full layer/module breakdown, the cross-cutting
behaviours, and the architectural constraints (the `CR-`/`NFR-` requirements). The summary below is
just enough to orient; consult the architecture doc when you need detail.

`src/`-layout package under `src/cpm_fm/`. `app.py:MainWindow` (a `QMainWindow` subclass) is the hub
that owns all components and wires UI events to them; `app.py:main` creates the `QApplication`, applies
the theme, shows the window, and runs `app.exec()` (entry point for both launchers). Three layers are
intentionally decoupled from the GUI so they are unit-testable without a running Qt app — **CR-014
forbids GUI-toolkit imports in `terminal/` and `utils/`**:

- `terminal/` — `serial_manager.py` (`SerialManager`: two `pyserial` ports, terminal + transport, with
  a daemon `_read_loop`), `xmodem.py` (`XModem`: hand-rolled X-Modem, blocking, on worker threads),
  `cpm_parser.py` (`CPMParser.parse_dir_output`: pure `DIR`-output scraper, the most-tested logic).
- `utils/` — `config_handler.py` (JSON settings load/save), `i18n.py` (the `tr(key)` translation
  singleton; strings in `lang/lang_<language>.txt`, English the reference/fallback), `file_filter.py`
  (pure filter/sort), `transfer_history.py` (`TransferHistory`: GUI-free JSON history store).
- `gui/` — Qt-only widgets/dialogs: `theme.py`, `terminal_window.py`, `config_dialogs.py`,
  `file_list_widget.py`, `transfer_dialog.py`, `conflict_dialog.py`, `filename_validation_dialog.py`,
  `transfer_history_dialog.py`, `manual_dialog.py`, `file_action_dialog.py`, `about_dialog.py`,
  `dialog_buttons.py`, `window_state.py`.

**Most safety-critical rule — the threading model (NFR-001/NFR-004):** serial reads and both transfer
directions run off the Qt GUI thread on daemon threads; any UI update from them must be marshalled onto

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [turbo-gecko/CPM_FM](https://github.com/turbo-gecko/CPM_FM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
