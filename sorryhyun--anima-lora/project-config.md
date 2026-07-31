---
trigger: always_on
description: Guidance for the PySide6 (Qt6) desktop GUI. Scoped to `gui/` — read the root `CLAUDE.md` for the training/config/daemon contracts this GUI drives. ~18.8k lines across 44 Python files; the two big tabs (`tabs/config_tab.py` 1739, `tabs/preprocess_tab.py` 1878) hold most of the surface (module-layout cleanup complete — history in `_archive/proposals/gui_refactor_plan.md`; `tabs/image_tab.py`'s autotag worker lives in `tabs/_autotag.py`).
---

# gui/CLAUDE.md

Guidance for the PySide6 (Qt6) desktop GUI. Scoped to `gui/` — read the root `CLAUDE.md` for the training/config/daemon contracts this GUI drives. ~18.8k lines across 44 Python files; the two big tabs (`tabs/config_tab.py` 1739, `tabs/preprocess_tab.py` 1878) hold most of the surface (module-layout cleanup complete — history in `_archive/proposals/gui_refactor_plan.md`; `tabs/image_tab.py`'s autotag worker lives in `tabs/_autotag.py`).

## What it is

A thin **front-end over the existing pipeline** — it edits TOML configs and submits jobs to the daemon; it does **not** contain training/torch logic. `config_io.py` and `_paths.py` are deliberately **Qt-free** (no PySide6 import) so they stay headless-unit-testable; keep them that way. The only `library/` imports are torch-free leaves (`library.config.dataset_keys`, `library.config.io`, `library.datasets.path_filter`) — don't pull torch/cv2-importing modules into the GUI or you slow startup by seconds (verify with `python -X importtime -c "import gui.app"`; torch must not appear).

## Launch

- `make gui` → `tasks.py gui` → `scripts/tasks/gui.py::cmd_gui` → `python -m gui`.
- `python -m gui` → `gui/__main__.py` → `gui/__init__.py::main` → `gui/app.py::main`.
- `app.py::main` (gui/app.py:377): `load_language()` → `ensure_daemon_quietly()` → build `MainWindow` → Qt loop.
- The legacy CLI `make lora-gui GUI_PRESETS=<variant>` is a *training* entry (runs `gui-methods/` configs directly); it is not this GUI. The GUI submits to the daemon instead.

## Architecture

- **`app.py::MainWindow`** — top bar (Guidebook / Models / Update / Queue + TensorBoard overlay toggles / ⚙ Settings at top right — language + the checkout-specific MCP registration (`claude mcp add` line + generic `mcpServers` JSON) via `SettingsDialog`, now in `settings_dialog.py`; a language change offers an immediate in-place window rebuild (`_reload_ui`) instead of requiring an app restart) + one tab set (Config = MethodsTab picker over the LoRA family + the promoted Turbo distiller, Preprocess, Dataset, Merge, Experimental = MethodsTab picker over the research methods + soup pipeline, EasyControl) in a `QStackedWidget` with the global TensorBoard and Queue overlay views. Dark `QPalette` via `_dark()`. `GuidebookDialog` + `_guidebook_path` live in `dialogs.py`; `_prefer_cleartype_font_engine` lives in `theme.py` next to `_load_bundled_fonts`.
- **Tabs** (`gui/tabs/`) inherit `LazyTabMixin` (first directory scan deferred to first view). On top of that, every tab after Config/Preprocess (Dataset, Merge, Experimental, EasyControl) sits behind a `widgets.LazyTabHolder` in `app.py` — the whole widget tree is only *constructed* on first open, so launch builds just the Config `MethodsTab` (eager inner `ConfigTab`), `PreprocessingTab` (the Train auto-chain needs it), and the TensorBoard panel; the daemon + Train button are live immediately. Consequence: a lazy tab's `ConfigTab._try_reattach` runs on first open, not at launch (the Queue overlay is the global job view either way), and code reaching into a lazy tab from outside must go through the holder's `.inner` (None until built — see `MainWindow._reload_image_tab_kb`). `MethodsTab` is a plain `QWidget` wrapper: a Method dropdown over an inner `ConfigTab` (flat `train.py --method` methods) + the distill editors (`TurboTrainTab` / soup) in a `QStackedWidget`; it backs both the **Config** tab (LoRA family + Turbo) and the **Experimental** tab (research methods + soup). `EasyControlTab` extends `ConfigTab` (inherits its dirty/observer machinery); `_DistillConfigTab` (the distill editors' base) is standalone and **lazy** — its TOML scan + `_try_reattach` run from `_lazy_init` on first show. The config-style tabs compose two shared mixins instead of re-implementing: `DirtyTrackingMixin` (`widgets/mixins.py`) + `DaemonJobMixin` (`_job_mixin.py`) — mix them in **before** `LazyTabMixin`/`QWidget`. A tab keeps only what genuinely differs (busy-UI, `_on_job_finished` teardown, log sink); don't copy the dirty/submit/poll boilerplate back into a tab.
- **`config_io.py`** — config discovery + merge + lint, all pure TOML/pathlib. `merged_gui_variant_preset(variant, preset)` returns `(dict, origin_map)` (origin_map = which key came from base/preset/variant). Variants are **auto-discovered** from `configs/gui-methods/*.toml` `[variant]` blocks (`family`/`order`) — adding a variant is one new file, no map to update; custom ones live in `gui-methods/custom/`. The **Hardware dropdown** (ConfigTab top bar) feeds the `preset` axis: options come from `list_hardware_presets()` (presets.toml sections tagged `[<name>.gui] group="hardware"`), the choice persists machine-wide in `gui_settings.json` (`hardware_preset`), and `ConfigTab._current_preset()` threads it through every merge/save/submit — variant files must not pin the hardware keys (method beats preset).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sorryhyun/anima_lora](https://github.com/sorryhyun/anima_lora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
