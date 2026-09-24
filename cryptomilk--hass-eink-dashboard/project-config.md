---
trigger: always_on
description: uv run --group test pytest                    # run all tests
---

# AGENTS.md

## Commands

```bash
uv run --group test pytest                    # run all tests
uv run --group test pytest tests/test_render.py::TestRenderWeather::test_weather_draws_temperature  # run a single test
uv run --group lint ruff check .              # lint
uv run --group format ruff format --check .   # format check
uv run --group typecheck ty check             # typecheck
uv run --group interrogate interrogate -vv custom_components/eink_dashboard/  # docstring coverage
```

After making changes, ALWAYS run:
- `uv run --group format ruff format . && uv run --group lint ruff check . && uv run --group format ruff format --check . && uv run --group typecheck ty check && uv run --group test pytest` for Python changes
- `pnpm --dir custom_components/eink_dashboard/frontend typecheck && pnpm --dir custom_components/eink_dashboard/frontend test` for TypeScript changes

## Architecture

This is a Home Assistant custom component (`custom_components/eink_dashboard/`)
that renders e-ink dashboard images as PNG bytes and exposes them via an HA
image entity and a public HTTP endpoint.

**Key files**:
- `render.py` — rendering orchestrator and shared helpers (delegates to
  `svg_render.py`; also holds `WidgetMetrics`, `_compute_metrics`,
  `color_to_hex`, `DEFAULT_METRICS`, `_load_font`, and other utilities
  imported lazily by widget modules and `widgets/_helpers.py`)
- `svg_render.py` — SVG rendering pipeline: Jinja2 templates, icon-inlining
  filters (`_mdi_svg_filter`, `_weather_svg_filter`), `render_widget_svg()`,
  `_compose_svg()`, `_svg_to_png()` via `resvg_py`
- `widgets/` — per-widget SVG context builders (`_build_*_context()`),
  one module per widget type; re-exported via `widgets/__init__.py`
- `widgets/_helpers.py` — shared layout helpers for widget builders:
  `_color_context()`, `_widget_dim()`, `_card_insets()`, `_metrics_context()`,
  `_title_layout()`, `_auto_row_height()`, `_fmt()`, `_entity_info_context()`,
  `_ACTIVE_STATES`
- `image.py` — `EinkDashboardImage` (`ImageEntity`), scheduled refresh, ETag tracking
- `http.py` — unauthenticated HTTP view at `/api/eink_dashboard/{entry_id}/image.png` with ETag/304 support
- `store.py` — `EinkDashboardStore`, persists widget list via HA's `Store` (`eink_dashboard.{entry_id}`)
- `config_flow.py` — multi-step config flow: name/dimensions/interval, plus TRMNL webhook management (add/remove named webhook targets)
- `optimize.py` — `optimize_for_eink(img, config)`: optional post-render pipeline (autocontrast, sharpness, contrast, grayscale level quantization)
- `push.py` — `async_push_image(session, url, image_bytes)`: HTTP POST of PNG bytes to a webhook URL
- `sensor.py` — `EinkDashboardSensor`, exposes dashboard state as an HA sensor entity
- `const.py` — enums, shared constants, and defaults

**MDI icons**: MDI icons are resolved at runtime by `_mdi_svg_filter()`
in `svg_render.py` via a two-stage lookup:
1. **`hass_frontend` (production)** — the `hass-frontend` pip package
   ships `static/mdi/iconMetadata.json` (a list of chunk descriptors)
   and per-chunk JSON files mapping icon names to SVG `d` path strings.
   `_load_hass_mdi_metadata()` reads the metadata and `_resolve_mdi_path()`
   uses `bisect` to find the right chunk for a given name.
2. **npm `@mdi/svg` (development / testing fallback)** — individual SVG
   files in `frontend/node_modules/@mdi/svg/svg/` (pnpm top-level symlink).
   Used automatically when `hass_frontend` is not installed.
No curated icon subset is needed; all 7 400+ MDI icons are available
through these sources without copying files manually.

**Rendering entry point**: `render_dashboard(widget_list, config) -> bytes` in `render.py`
- `config` is a `DisplayConfig` dict with `width`, `height`, `rotation`, and `states` (HA entity ID → state dict)
- Default display: 758×1024 px, 8-bit grayscale (`"L"` mode)
- Dispatches each widget to its SVG context builder via `_SVG_RENDERERS` in
  `svg_render.py`, composes one root SVG, rasterises with `resvg_py`, then
  applies rotation and e-ink optimisation, and returns PNG bytes

**Widget types** (`WidgetType` in `const.py`): `ENTITY`, `HEADING`,
`SEPARATOR`, `TILE`, `WEATHER`, `DEVICE_BATTERY`, `WASTE_SCHEDULE`.
Deprecated types (hidden from the widget picker but kept for existing
configs): `TEXT` (superseded by `HEADING`), `SENSOR_ROWS`,
`STATUS_ICONS`.

**Adding a widget type**:
1. Add the new value to `WidgetType` in `const.py`
2. Create `templates/foo.svg.j2` (may import `_macros.svg.j2` helpers)
3. Write `_build_foo_context(widget, config) -> dict` in `widgets/foo.py`
4. Re-export from `widgets/__init__.py` and register in
   `_SVG_RENDERERS` in `svg_render.py`
5. Add the widget type to `frontend/src/types/ha.d.ts`
6. Add the schema and entry to `WIDGET_TYPES` in `frontend/src/eink-dashboard-editor.ts`

**Converting or redesigning a widget type** (PIL→SVG migration or new
widget):
Do not write code directly. Invoke the three skills in order using the
`Skill` tool — each one must complete before the next is called:
1. `/implement-widget-tests` — write failing tests (TDD red phase)
2. `/implement-widget` — implement the SVG template and Python
   context builder (green phase)
3. `/implement-widget-frontend` — TS types and editor schema


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cryptomilk/hass-eink-dashboard](https://github.com/cryptomilk/hass-eink-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
