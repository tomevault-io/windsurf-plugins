---
trigger: always_on
description: `netbox-reorder-rack` is a NetBox plugin that adds a drag-and-drop rack elevation. Devices are repositioned by dragging them between units, between the front and rear faces, and in and out of a non-racked list; on save, the arrangement is written back as ordinary `Device.position` and `Device.face` values.
---

# AGENTS.md — netbox-reorder-rack

## Repository Overview

`netbox-reorder-rack` is a NetBox plugin that adds a drag-and-drop rack elevation. Devices are repositioned by dragging them between units, between the front and rear faces, and in and out of a non-racked list; on save, the arrangement is written back as ordinary `Device.position` and `Device.face` values.

The plugin has **no models, no migrations, and no configuration parameters**. It consists of a `PluginTemplateExtension` (the Reorder button), one view that renders the elevation, one REST endpoint that saves it, a set of template filters, and a bundled browser-side editor. The supported NetBox range is in `COMPATIBILITY.md`.

Because it stores nothing of its own, all of its behaviour is *read NetBox state → rearrange in the browser → write device positions back*. There is no plugin-owned data to get out of step.

## Tech Stack

- Python — no version declared in `setup.py`; NetBox's own floor applies
- NetBox (host app) — **`min_version` is set to `"4.7.0"`; no `max_version`.** NetBox refuses to load the plugin below 4.7. `COMPATIBILITY.md` records the full tested ranges, including earlier releases that supported older NetBox versions
- Django + Django REST Framework (provided by NetBox)
- [Gridstack](https://gridstackjs.com/) **12.6.0** and [Bootstrap](https://getbootstrap.com/) **5.3.8** — bundled, pinned to the versions NetBox ships
- esbuild **0.28.2** — bundler, a devDependency. **yarn only**; see Front-end below
- NetBox's test framework (`utilities.testing.TestCase`), run through `manage.py test`
- pre-commit (`.pre-commit-config.yaml`) with djlint for templates (`.djlintrc`)

The plugin declares **no** `install_requires`. All front-end versions are pinned exactly — no `^` or `~`.

## Repository Map

```text
.
├── netbox_reorder_rack/
│   ├── __init__.py              — PluginConfig. No default_settings; sets min_version = "4.7.0".
│   ├── template_content.py      — ReorderButton (PluginTemplateExtension) on dcim.rack.
│   ├── views.py                 — ReorderView: registered on Rack via register_model_view,
│   │                              renders the elevation from rack.get_rack_units().
│   ├── urls.py                  — Wires the model view through get_model_urls('dcim','rack').
│   ├── api/views.py             — SaveViewSet: validates and writes device positions in one
│   │                              transaction. Not a NetBoxModelViewSet; schema is disabled.
│   ├── api/urls.py              — NetBoxRouter, registered as 'save'.
│   ├── templatetags/rack.py     — Filters used by the elevation template.
│   ├── templates/netbox_reorder_rack/
│   │   ├── rack.html            — Thin shell: extends generic/object.html, supplies only the
│   │   │                          CSS/JS asset blocks. Page body comes from the view's layout.
│   │   ├── inc/reorder.html     — The elevation, rendered as a TemplatePanel.
│   │   ├── inc/rack_elevation.html — One elevation (front or rear).
│   │   └── inc/rack_button.html — The Reorder button.
│   ├── static/netbox_reorder_rack/  — BUILT OUTPUT, committed. Do not hand-edit.
│   │   ├── js/rack.js (+ .map)
│   │   └── css/rack.css
│   ├── static_dev/              — Front-end sources and the bundler.
│   │   ├── js/rack.js           — Edit this, then rebuild.
│   │   ├── css/rack.css         — Plain CSS. No Sass.
│   │   ├── bundle.js            — esbuild build script.
│   │   ├── package.json
│   │   └── yarn.lock            — The only lockfile. Do not add package-lock.json.
│   └── tests/                   — test_view.py, test_api.py. Need PostgreSQL via NetBox.
├── docs/                        — mkdocs site (see mkdocs.yml for nav).
├── CHANGELOG.md                 — Canonical changelog; docs/changelog.md includes it.
├── COMPATIBILITY.md             — Plugin release to NetBox version matrix.
└── .github/workflows/
    ├── pre-commit.yml           — Runs pre-commit. The test suite does NOT run in CI.
    └── python-publish.yml       — Publishes to PyPI.
```

## Architecture

### The round trip

```
Rack detail view
  └─ ReorderButton (template_content.py) → dcim:rack_reorder
       └─ ReorderView (views.py) — a generic.ObjectView with a declarative layout
            get_extra_context() supplies rack.get_rack_units(expand_devices=False, face=...)
            for both faces plus non-racked devices, to inc/reorder.html via a TemplatePanel
              └─ static/.../rack.js (Gridstack) rearranges in the browser
                   └─ PUT /api/plugins/reorder/save/<rack_pk>/
                        └─ SaveViewSet writes Device.position / Device.face
```

`expand_devices=False` matters: it returns only the bottom-most unit for a multi-U device, with a `height` attribute, which is what the grid needs. Passing `True` would repeat the device once per unit.

### Saving


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netbox-community/netbox-reorder-rack](https://github.com/netbox-community/netbox-reorder-rack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
