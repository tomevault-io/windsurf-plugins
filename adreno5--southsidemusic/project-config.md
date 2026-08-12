---
trigger: always_on
description: SouthsideMusic is a Windows-only PySide6 desktop client for NetEase CloudMusic:
---

# AGENTS.md - SouthsideMusic

## Project Overview

SouthsideMusic is a Windows-only PySide6 desktop client for NetEase CloudMusic:
streaming playback, word-by-word lyrics, loudness normalization, desktop lyrics,
local favorites, song export, and auto-update support.

The app is packaged with Nuitka and distributed through Inno Setup. Runtime caches
live under `data/`; persistent user settings live in `config.json`.

Primary docs are `docs/README.md` and `docs/README_zh.md`. No Cursor rules
(`.cursor/rules/` or `.cursorrules`) and no Copilot instructions
(`.github/copilot-instructions.md`) exist at the time this file was written.

## Environment

- Target OS: Windows.
- Project metadata in `pyproject.toml` says Python `>=3.13`; prefer that over
  older docs that mention Python 3.12+ / 3.12.7.
- `uv.lock` is present; prefer `uv run ...` when available.
- Initial workspace setup is automated by `python setup_workspace.py`.
- Build output goes to `build.result\raw\` and optionally `build.result\installer\`.

## Commands

```bash
python setup_workspace.py                 # bootstrap dependencies/tooling
uv run src/main.py                        # run from source (preferred)
python src/main.py                        # run if environment is already active
build.bat                                 # full Windows build/package flow
python scripts/create_icon.py             # regenerate icons/app.ico

uv run ruff check .                       # lint all files
uv run ruff format --check .              # check formatting
uv run ruff format .                      # format files
uv run mypy src/                          # type check source tree
python -m py_compile src/main.py          # quick syntax/import smoke check
```

`build.bat` deletes old outputs, runs Nuitka on `launcher.py`, copies embedded
Python/resources/source, regenerates the icon, then runs Inno Setup if `ISCC.exe`
is installed. Without Inno Setup, raw portable files remain in `build.result\raw\`.

## Tests

There is no formal test suite yet. `src/test.py` is a manual API exploration
script, not pytest. Do not invent a test framework unless explicitly needed.

If tests are added, use these commands:

```bash
uv run python -m pytest tests/                     # all tests
uv run python -m pytest tests/test_foo.py          # one test file
uv run python -m pytest tests/test_foo.py -k name  # one test by expression
uv run python -m pytest tests/test_foo.py::test_x  # one exact test
```

For small non-test changes, prefer narrow validation first: `python -m py_compile <file>`, then `uv run ruff check <file>`, then broader lint/type checks if useful.

## Project Structure

```text
src/
  main.py          # app entry, QApplication setup, logging, excepthook
  imports.py       # centralized imports/re-exports for Qt, typing, events
  core/            # audio, config, models, lyrics, theme, icons, backends
  services/        # event bus and update checks
  views/           # PySide6 UI pages, cards, windows, widgets
  pyncm/           # forked NetEase CloudMusic API client
docs/              # English/Chinese user documentation
data/              # runtime caches for music, images, lyrics, temp data
icons/, images/    # packaged UI resources
fonts/             # bundled HarmonyOS Sans SC font assets
config.json        # hand-editable persisted user config
```

Reference style files: `src/views/search_page.py`, `src/views/error_popup.py`.

## Import Style

- Use `from __future__ import annotations` when the file already follows it.
- Import Qt/PySide6 classes from `imports`, not directly from PySide6:

```python
from imports import QTimer, QVBoxLayout, QWidget, Qt, Signal, event_bus
```

- `src/imports.py` re-exports PySide6 classes, typing helpers, qfluentwidgets,
  and event bus members.
- Direct third-party imports are fine for non-Qt libraries (`numpy`, `requests`).
- Use `if TYPE_CHECKING:` for type-only imports that could create circular imports.
- Keep imports grouped as standard library, third-party, then project imports.
- qfluentwidgets may be imported directly when existing code does so.

## Formatting

- Ruff config is `.ruff.toml`: line length 88, indent width 4, single quotes.
- Keep edits ASCII unless existing content or UI copy requires non-ASCII.
- Keep QSS color names lowercase (`'white'`, `'black'`).
- Prefer small, local diffs. Do not reformat unrelated files.
- Avoid large abstractions; this codebase favors direct PySide code.
- Add comments only for non-obvious behavior; keep them English and sparse.

## Types

- Annotate all parameters and return types in new or changed functions.
- Use PEP 604 unions (`str | None`) except when preserving existing `pyncm/` style.
- Use `@dataclass` for config/data containers.
- Use `ABC` / `@abstractmethod` for explicit backend interfaces only.
- Use `cast()` for fields populated after construction when needed.
- Use `@override` where parent methods are intentionally overridden.
- Keep public docstrings short: `"""single line."""`.

## Naming

- Files/modules: `snake_case.py`.
- Classes and Qt widgets: `PascalCase` (`AudioPlayer`, `SearchPage`).
- Public methods: `camelCase` (`getValue`, `saveConfig`, `loadFavorites`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Adreno5/SouthsideMusic](https://github.com/Adreno5/SouthsideMusic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
