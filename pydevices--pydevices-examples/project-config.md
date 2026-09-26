---
trigger: always_on
description: This repo is a **showcase**: examples and a PyScript gallery for the PyDevices
---

# AGENTS.md

## Cursor Cloud specific instructions

This repo is a **showcase**: examples and a PyScript gallery for the PyDevices
driver stack, not the product itself. The shareable product libraries and their
docs live in sibling `pydevices`. There is no build step for examples.
`.site/pyscript/lib` is a symlink to `../../lib`, so editing `lib/` updates the
PyScript gallery too.

**Docs map** (this repo's `docs/` is small on purpose — see
[`docs/README.md`](docs/README.md)): [`docs/peterhinch-guis.md`](docs/peterhinch-guis.md)
(Nano-GUI / Micro-GUI / MicroPython-Touch), [`docs/tft-gui.md`](docs/tft-gui.md)
(@russhughes TFT GUI stub), [`docs/screenshots/`](docs/screenshots/README.md).
Code-adjacent runbooks: [`tools/README.md`](tools/README.md) (example test
matrix, PyScript/Playwright debugging, LVGL timer harnesses),
[`lib/utils/README.md`](lib/utils/README.md), [`scripts/README.md`](scripts/README.md),
[`tests/README.md`](tests/README.md).

### Environment

`displaydev`, `audiodev`, optional `appdev`, `multimer`, `events`, `keys`,
and portable hardware utilities live in sibling
[pydevices](https://github.com/PyDevices/pydevices), which
also owns TestPyPI/MIP publishing. Non-LVGL examples instantiate
`appdev.App(board_config)`; LVGL examples import it from `display_driver`.
Board configs never own an `appdev.App`. `AutoDisplay` is imported from
`displaydev.auto` only.

- **Cursor Cloud (multi-repo workspace):** do not use a local
  `.cursor/environment.json` in this repo. The canonical cloud environment lives
  in [PyDevices/.github](https://github.com/PyDevices/.github) — start Cloud
  Agents from that repo with the saved **Pydevices Cloud
  Workspace** environment. Its install command is
  `bash scripts/cloud-workspace-install.sh` (relative to the `.github` checkout),
  which symlinks `/agent/repos/*` into `~/gh/pydevices/`. See
  [AGENTS.md there](https://github.com/PyDevices/.github/blob/main/AGENTS.md).
- Use the repo-root virtualenv at `.venv` for all Python tooling
  (`.venv/bin/python`, `.venv/bin/ruff`, `.venv/bin/pytest`). The system `python3` has no project
  dependencies installed.
- **Browser testing with Playwright:** see
  [`.agents/rules/playwright_environment_rule.md`](../.agents/rules/playwright_environment_rule.md).
- Desktop matrices use repo `.venv` (`cpython-venv`) plus interpreters on
  `PATH` (`micropython`, `circuitpython`, and when present
  `micropython.exe` / `python.exe`). `jupyter.py` (from `pydevices/bin`), `./scripts/pyscript.sh`, and
  `android.py` (from `pydevices/bin`, on `PATH`) aid
  Jupyter, PyScript, and Android (adb stage onto `org.pydevices.runner`;
  cwd paths like CLI Python — not PyScript gallery). Opt-in matrix:
  `tools/example_test_kit.py --only-interpreter android …`.
- The desktop display backend on CPython on Windows is `PGDisplay` (pygame-ce;
  `import pygame`). Prefer `python.exe` for PG work. Do **not** install pygame-ce
  into `.venv` / system `python3` on this laptop — those stay SDL-primary;
  `board_config` falls back to `SDLDisplay` when pygame-ce's public
  `pygame.Window` API is missing. `pygame-ce` is intentionally not in
  `requirements-dev.txt`.

### Tests and lint

- Unit tests (stdlib `unittest`, no third-party runner needed):
  `.venv/bin/python -m unittest discover -s tests`
- Lint/format: `.venv/bin/ruff check --force-exclude --extend-exclude '*.ipynb' lib tests tools`
  and `.venv/bin/ruff format --check --force-exclude --extend-exclude '*.ipynb' lib tests tools`.
  Note `pyproject.toml` **excludes `lib/examples/**`**
  (and a few others) from ruff, so example files are not linted/formatted; do not
  be surprised when `ruff format --check` on an example path reports a diff.
- The pre-commit hooks (`.pre-commit-config.yaml`) are `ruff-check`,
  `ruff-format` (python/pyi only), and `nbstripout` for notebooks. `ruff` does
  **not** lint `*.ipynb` under the hook config, so pre-existing notebook findings
  from `ruff check` on the whole tree can be ignored.

### Running examples headlessly (GUI smoke tests)

- **Read [`tools/README.md` — Example test matrix](tools/README.md#example-test-matrix)
  first** — agent runbook for the cross-interpreter example test system. Canonical
  interpreter list: [`tools/example_interpreters.toml`](tools/example_interpreters.toml);
  per-example metadata: [`tools/example_test_manifest.toml`](tools/example_test_manifest.toml).
- **Preferred thorough gate:** example-by-example with **all selected interpreters
  in parallel** (`--jobs 0`): **5** desktop for sync, **7** for async; both
  `PYDEVICES_TIMER_ASYNC=0` and `=1`, `--fail-fast`, line-buffered live log,
  fix after a failed example wave then resume — see
  [Preferred method](tools/README.md#preferred-method-parallel-interpreters-fail-fast-both-timer-modes)
  and [Windows PE under WSL](tools/README.md#windows-pe-under-wsl).
  Do **not** forward `SDL_*` to `*.exe` (PE windows should appear; unix stays
  headless from the shell export). A PE `hang` with a live window means quit
  failed, not that PE failed to start. `--curated-only` is smoke only.
- `--only-example` / `--only-interpreter` take **space-separated** ids on one flag
  (`--only-interpreter circuitpython python.exe`). Repeating the flag keeps only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PyDevices/pydevices-examples](https://github.com/PyDevices/pydevices-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
