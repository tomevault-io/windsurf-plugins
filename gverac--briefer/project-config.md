---
trigger: always_on
description: Guidance for working in this repo.
---

# CLAUDE.md

Guidance for working in this repo.

## What this is

A daily-brief printer. It builds a short briefing (weather, birthdays,
reminders, …) and prints it on a 58mm ESC/POS thermal receipt printer driven by
[python-escpos](https://github.com/python-escpos/python-escpos).

## Where it runs

- **Target hardware:** Raspberry Pi Zero 2 W (arm64, quad-core) on Raspberry Pi
  OS Bookworm + NetworkManager. The printer is attached via USB/serial. arm64
  means prebuilt wheels are available for everything (incl. `anthropic`'s
  pydantic-core), so all deps in `requirements.txt` install normally.
- **Development:** done on a laptop (macOS) using the `dummy` printer backend,
  then deployed to the Pi. Keep all hardware access behind `daily_brief.printer`
  so the rest of the code stays runnable and testable without a printer.
- Python 3.11+ (uses stdlib `tomllib`).

## Architecture

The brief is rendered as **one tall bitmap** (modern TrueType font, thin rules,
checkboxes, weather pictograms) and sent to the printer as a raster image — not
with the built-in ESC/POS text font. That's a deliberate choice: it's the only
way to get a non-receipt look and graphics. The Pi Zero W rasterizes this more
slowly than text, but it's a once-a-day print.

**Data model: section (block) → brief (ordered set) → schedule (time → brief).**
One `config.toml` holds global tables plus `[[briefs]]` (each with nested
`[[briefs.sections]]`) and `[[schedules]]`.

- `daily_brief/config.py` — `load_config()` reads TOML into dataclasses
  (`BriefConfig`, `ScheduleConfig`, `NetworkConfig`, `WebConfig`, …); a legacy
  flat `[[sections]]` file migrates to one "default" brief. `save_config()`
  writes TOML via `tomli_w` (the setup UI uses it; comments aren't preserved, so
  `config.example.toml` stays the documented template). `Config.brief(name)`
  looks one up. Section `options` is a free per-source dict.
- `daily_brief/printer.py` — **the only module that touches hardware.**
  `make_printer(cfg)` returns a python-escpos device (Dummy / Usb / Serial).
- `daily_brief/brief.py` — data model: `Brief` → `Section` → `Item`s
  (`Text`, `Checkbox`, `Bullet`, `Banner`, `KeyVal`, `Weather`, `Picture`,
  `ProgressBar`, `Mono`, `Title`). `Section.icon` is a header pictogram key; `Section.bare` renders items
  with no rule/heading (used by the `greeting` section's centered `Title`). The
  header is just the first section now — there's no special header rendering.
  Sources are data-only; `build_brief(config, brief, ...)` iterates the brief's
  sections.
- `daily_brief/render.py` — `Canvas` (PIL) + `render_brief(printer, brief, cfg)`.
  Draws each Item, crops to height, prints via `printer.image(...)`. `printer`
  may be `None` to only write a PNG preview. No special header — the greeting is
  just the first (bare) section.
- `daily_brief/sources/` — one builder per source, registered in
  `sources/__init__.py` (`BUILDERS`); `build_section` runs each through
  `safe_build`, which turns any failure into an "(unavailable)" section. Header
  icons are off by default except birthdays (`DEFAULT_SECTION_ICONS`); any section
  can opt in with `icon = "<key>"` or out with `icon = ""`. `_http.py` wraps
  `requests` with a timeout + TTL file cache (`~/.cache/daily_brief/`) and stale
  fallback.
  Space sources live in `space.py` (iss/moon/planets). The `ascii` source draws
  a daily piece from `daily_brief/ascii_art.py` as a `Mono` item, or has Claude
  draw it when `use_claude = true`. The `ai` source (`ai.py`) feeds a config
  `prompt` to Claude and prints the answer, capped to `max_chars`; with
  `use_web_search = true` it grants Claude the Anthropic-hosted web search tool
  (`llm.WEB_SEARCH_TOOL`) so the answer can use current info — run server-side,
  resuming the tool loop on `pause_turn`, no local fetching.
- `daily_brief/assets/` — bundled Inter + DejaVu Sans Mono fonts, weather
  pictograms (`weather/`), header/banner icons (`icons/`), and the ISS world map
  (`space/world.png`). Regenerate icons with `scripts/gen_icons.py` /
  `scripts/gen_weather_icons.py`.
- `daily_brief/llm.py` — Claude wrapper (`generate()`). `[claude] enabled` is the
  master toggle; `ClaudeConfig.active` (= `enabled and api_key`) gates everything.
  AI users: the **ai** section (gated by `active`), and **greeting** + **word** +
  **ascii** (each has its own per-section `use_claude`, so AI runs only when
  `use_claude and active`). **When a section's AI call fails it surfaces "(AI
  unavailable)" rather than silently using its local version**; with AI off /
  unkeyed / `use_claude` unchecked, it uses local behavior (rotation / Free
  Dictionary / gallery). The `greeting` source (`sources/greeting.py`) renders the
  centered header (greeting + `date_format` line) and offers `style` presets.
  Each section type's editable fields are described in `sources/specs.py`
  (`SECTION_SPECS`), which drives the web UI's forms + validation.
- `daily_brief/__main__.py` — print CLI (`python -m daily_brief --brief <name>`);
  `--dry-run` writes a PNG preview, `--out` saves the bitmap, `--backend` overrides.
- `daily_brief/daemon.py` — the long-running service (`python -m daily_brief.daemon`).
  `Scheduler` fires schedules at their time (reloads `config.toml` on change);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gverac/briefer](https://github.com/gverac/briefer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
