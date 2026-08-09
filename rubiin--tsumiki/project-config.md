---
trigger: always_on
description: > Precedence: if an explicit user instruction in a session conflicts with this
---

# Claude Configuration for Tsumiki

> Precedence: if an explicit user instruction in a session conflicts with this
> file, the user instruction wins for that task. Otherwise, follow this file.

## 1. Project Overview

Tsumiki is a modular status bar / panel for Hyprland, built with
[Fabric](https://github.com/Fabric-Development/fabric) (Python + GTK3 widget
framework). Main code is Python; styles are SCSS; docs are an Astro site.

## 2. Repository Layout

| Path | Contents |
|------|----------|
| `modules/` | Core UI modules (dock, overview, notification, settings, etc.) |
| `services/` | Background services (battery, weather, network, brightness, etc.) |
| `widgets/` | Individual widget implementations |
| `shared/` | Reusable UI components and mixins |
| `styles/` | SCSS stylesheets and theming |
| `utils/` | Utility functions, config management, helpers |
| `docs/` | Astro-based documentation site |
| `tests/` | Test suite |
| `assets/` | Static assets (icons, images, i18n, emoji, etc.) |

## 3. Key Files

- `main.py` — entry point
- `pyproject.toml` — Python project metadata, targets Python 3.12 style checks
- `config.toml` — configuration example / user config
- `tsumiki.schema.json` — configuration schema (source of truth for config shape)
- `Justfile` — task recipes

## 4. Setup, Build & Test Commands

```bash
./init.sh -setup          # install/setup dependencies
./init.sh -start           # run the app
just --list                 # list all available task recipes
python -m pytest tests/     # run the test suite
cd docs && pnpm install && pnpm build   # build the docs site
fabric-cli gs GtkLayerShell-0.1 Playerctl-2.0 NM-1.0  # regenerate gi stubs
```

The following are **one-off/maintainer commands** — run only when explicitly
requested, never as a side effect of an unrelated change:

- `./install.sh` (system-level bootstrap install)
- `pip freeze > requirements.txt` (overwrites the pinned dependency list)

## 5. Guardrails

- Do not run `./install.sh`, `pip freeze > requirements.txt`, or any
  destructive/system-modifying command unless the user explicitly asks for it.
- Do not revert user changes unless explicitly asked.
- Do not edit generated files (docs build output, stub files from
  `fabric-cli gs`) directly — regenerate them via their command instead.
- `tsumiki.schema.json` is the source of truth for config shape. Never let
  `config.toml` or `utils/config.py` drift from it silently — update all three
  together.
- Don't commit, push, or open a PR unless asked to.
- When in doubt about scope, prefer the smaller change and ask.

## 6. Before Starting Work

- Confirm whether the task is a **feature, bugfix, or refactor**.
- Check whether it touches the **public config schema** (`tsumiki.schema.json`).
- Decide whether it needs **new tests**.
- Decide whether it needs **doc updates** (README, docs site, or schema).
- For unfamiliar areas, skim `README.md`, `CONTRIBUTING.md`, and `doc.md` first.

## 7. Editing Conventions

- Prefer small, focused changes over broad rewrites.
- Keep style consistent with the surrounding code.
- Prefer refactors that reduce duplication and nested branching.
- Before touching popup, dock, notification, OSD, or settings UI code, check
  `shared/` for an existing helper first — don't reimplement.
- Settings UI is large and repetitive; prefer generic builders over copy-paste.
- Ruff is enabled — keep imports clean and avoid obvious lint noise.
- Use ASCII in code, comments, and identifiers unless the file already uses
  Unicode. This does **not** apply to intentional UI glyphs (e.g. Nerd Font /
  emoji icons in widget config defaults) — those are a design choice, not a
  style violation.

Architecture notes worth knowing before editing:
- Popup logic is centralized in `shared/popup.py`.
- Lazy widget loading happens in `modules/bar.py`.

## 8. Code Review Checklist

1. **Conventions** — structure, naming, and patterns match the rest of the repo.
2. **Integration** — minimal coupling, proper separation of concerns.
3. **Maintainability** — readable, documented, testable.
4. **Schema alignment** — respects `tsumiki.schema.json`.

## 9. Common Tasks

| Task | Approach |
|------|----------|
| Add new widget | See §10 |
| Add new module | See §11 |
| Add new service | Create in `services/`, register in `main.py` |
| Add config option | Update schema, add to `utils/config.py`, update docs |
| Style changes | Edit relevant SCSS, test theme variants |
| Fix UI bug | See §13 |

## 10. Adding a New Widget (panel-based, toggleable)

Widgets live in `widgets/` and are typically panel buttons that open popovers
or show quick info.

**Step 1 — Widget file** (`widgets/my_widget.py`):
```python
from fabric.widgets.box import Box
from fabric.widgets.label import Label

from shared.mixins import PopoverMixin
from shared.widget_container import ButtonWidget
from utils.widget_utils import nerd_font_icon


class MyWidgetMenu(Box):
    """Popover content."""

    def __init__(self, parent=None, **kwargs):
        super().__init__(
            name="my-widget-menu",
            orientation="v",
            spacing=8,
            **kwargs,
        )
        self._parent = parent
        self.children = [Label(label="Menu content")]

    def close(self, *_):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rubiin/Tsumiki](https://github.com/rubiin/Tsumiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
