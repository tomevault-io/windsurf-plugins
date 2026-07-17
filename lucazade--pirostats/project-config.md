---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Python daemon that renders KDE Plasma panel + tooltip system stats as HTML. It runs in memory and atomically writes `<runtime>/{panel,tooltip}.html` (`<runtime>` = `$XDG_RUNTIME_DIR/pirostats`, see `src/runtime.py`); the bundled Plasma applet (`plasmoid/`, id `com.github.lucazade.pirostats`) watches that directory and `cat`s the files when they change. **There is one clock in the system, `display.poll_interval`** — the applet has none, so nothing aliases and no frame ages waiting to be noticed. See `README.md` for the full user-facing story — this file covers only what you need to work productively in the code.

## Commands

```bash
# Run all tests
python3 -m pytest tests/ -v

# Run a single test file / test
python3 -m pytest tests/test_config.py -v
python3 -m pytest tests/test_formatter.py -k some_test_name

# Regenerate the golden HTML snapshots after an INTENDED render change
UPDATE_GOLDEN=1 python3 -m pytest tests/test_golden_render.py
# (goldens live in tests/golden/{panel_h,panel_v,tooltip}.html)

# Dead code: the same sweep test_deadcode.py gates, run by hand for the report
vulture src/ tests/ pirostats tests/vulture_whitelist.py --min-confidence 60

# Lint: the same check test_lint.py gates (config in ruff.toml), run by hand
ruff check .

# One-shot diagnostics / render (no daemon, readable in terminal)
./pirostats probe --config config/config.toml      # every item, raw readings
./pirostats render                                 # HTML stripped to text
./pirostats render --component tooltip --format html   # -> /tmp/pirostats_render_tooltip.html
./pirostats render --component panel --layout horizontal|vertical  # force orientation
./pirostats render --page processes|cpu_cores|connections|fastfetch|graphs  # a tooltip deep-dive page (any page, even one not in pages.order)
./pirostats profiling --config config/config.toml  # per-item timing, cache state
./pirostats list-items                             # authoritative list of valid metric:form tokens

# Live daemon
systemctl --user status pirostats
journalctl --user -u pirostats -f
```

Run everything from the repo root; `./pirostats` prepends `src/` to `sys.path`, so there is no package install step in dev. `config.py`/`daemon.py` resolve `config/config.toml` relative to `__file__`, so the repo can live anywhere.

No build step — pure Python 3.11+ (stdlib `tomllib`), the test suite is the gate. Two of its checks shell out to optional dev tools and **skip when the tool is absent**, so a bare checkout still runs everything else: `vulture` (dead-code gate) and `ruff` (lint gate, config in `ruff.toml`). Both are run as tests, so `python3 -m pytest tests/` covers them — see Testing notes. `pacman -S vulture ruff` / `pip install vulture ruff`.

## Architecture: the metric × form model

The central idea is that an item is **not** a flat name but a pair `metric:form` (e.g. `cpu_usage:bar`, `hd_temp:pair`). Three files own the two axes and their dispatch — read all three together before touching item behavior:

- **`metrics.py`** — the *what* axis. A metric declares only what's intrinsic and form-independent: `needs` (which sensor helper feeds it), `gate` (when the hardware is present), `surfaces` (panel/tooltip eligibility), and which generic `forms` it supports. Glyph, label and thresholds are deliberately NOT here.
- **`forms.py`** — the *how* axis: `Form` enum (value/bar/spark/braille/pair/…), `Surface` flags, and `FORM_SURFACES` (which surfaces a form is valid on). A generic form declares no skeleton — how a row lays out is read off its cells by `mono_render._plan_row` (`docs/LAYOUT.md`); `Shape` exists only as the value of a metric's `intrinsic_shape`.
- **`registry.py`** — the dispatch `_RENDER[(metric, form)] -> GroupFn`. Regular entries are built from the declarative cell-factory library in **`items.py`** (`row`/`per` + `label`/`value`/`spark`/…); irregular ones (combos, string joins, batteries, adaptive layouts, own skeletons) are explicit exception functions `(f, ident, r, tooltip) -> list[Row]`. `registry.py` also holds the **token layer** (`parse` of `"metric[:form]"`) that formatter/config/sensors consume.

Placement is **derived, not declared**: an item's real surfaces = intersection of the form's `FORM_SURFACES` and the metric's `surfaces`. It's also **enforced**: `config._drop_misplaced_items` drops a token listed on a surface its surfaces don't admit (with a stderr warning), so `cpu_usage:spark` in a `[tooltip.*]` doesn't render a label-less trace there. `_drop_unknown_items` does the same for typos.

The two-axis identity flows through as an `Ident` (metric + form), which the cells turn into the CSS class `.item-<metric>.form-<form>`. The `BAR` form is orientation-adaptive: it renders as a vertical column in the horizontal panel and an inline bar in the vertical panel, picked in `registry._form_token` — this is also where `[panel_horizontal]`/`[panel_vertical]` config merges land.

## Rendering pipeline


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucazade/pirostats](https://github.com/lucazade/pirostats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
