---
trigger: always_on
description: `netbox-qrcode` is a NetBox plugin that renders printable QR code labels on object detail pages. A label combines a QR code (encoding the object's URL by default) with text drawn from the object's own fields, and its layout is defined entirely in configuration.
---

# AGENTS.md — netbox-qrcode

## Repository Overview

`netbox-qrcode` is a NetBox plugin that renders printable QR code labels on object detail pages. A label combines a QR code (encoding the object's URL by default) with text drawn from the object's own fields, and its layout is defined entirely in configuration.

The plugin is unusually thin for a NetBox plugin: it has **no models, no migrations, no views, no REST or GraphQL API, and no database access**. It consists of `PluginTemplateExtension` subclasses that render Django templates into the right-hand or left-hand column of existing NetBox detail views. Everything a user can change lives in `PLUGINS_CONFIG`.

Labels are rendered for `dcim.device`, `dcim.module`, `dcim.rack`, `dcim.cable`, `dcim.location`, `dcim.powerfeed` and `dcim.powerpanel`, plus `netbox_inventory.asset` when [netbox-inventory](https://github.com/ArnesSI/netbox-inventory) is installed. The supported NetBox range is in `COMPATIBILITY.md` (4.7.0 – 4.7.x at the time of writing).

## Tech Stack

- Python (defer to `setup.py`; currently `>=3.12`)
- NetBox (host app — minimum and maximum versions are pinned in `netbox_qrcode/__init__.py` `min_version` / `max_version`; `COMPATIBILITY.md` summarises the matrix)
- Django (provided by NetBox — 6.0.x on NetBox 4.7). Used both to render the plugin's own templates and to render user-supplied `text_template` / `url_template` strings
- `qrcode` + `Pillow` — QR image generation, emitted as a base64 PNG data URI
- `packaging` — version comparison in `template_content.py`
- `unittest` (standard library) for tests — **not** pytest, and not `django.test.TestCase`
- mkdocs + mkdocs-material for user-facing docs

There is no linter configured in this repo. Defer all version pins to `setup.py` and `netbox_qrcode/__init__.py`.

## Repository Map

```text
.
├── netbox_qrcode/
│   ├── __init__.py                  — QRCodeConfig (PluginConfig): version pins and default_settings.
│   │                                  default_settings is the single source of truth for every
│   │                                  configuration parameter and its default.
│   ├── template_content.py          — PluginTemplateExtension subclasses, one per supported model.
│   │                                  Holds the per-label loop, error isolation, and config-key naming.
│   ├── template_content_functions.py— Label content helpers: config resolution, URL, text, QR.
│   ├── utilities.py                 — get_qr() / get_img_b64(): qrcode + Pillow wrappers.
│   ├── version.py                   — __version__ (read by setup.py and the relpatch target).
│   ├── templates/netbox_qrcode/
│   │   ├── qrcode3.html             — The label panel. All layout lives here as inline CSS.
│   │   ├── qrcode3_sub_qrcode.html  — The QR image block, {% include %}d by qrcode3.html.
│   │   ├── qrcode_error.html        — Shown in place of a label that failed to render.
│   │   └── qrcode.html              — Legacy NetBox 1/2 panel. Unreachable; see Architecture.
│   └── tests/                       — Standalone unittest suite; needs no database or NetBox.
├── docs/                            — mkdocs site (see mkdocs.yml for nav).
│   ├── index.md, installation.md, configuration.md, label-examples.md, printing.md
│   ├── changelog.md             — Snippet include of the root CHANGELOG.md; no content of its own.
│   ├── requirements.txt             — mkdocs-material, for building the docs.
│   └── img/                         — Screenshots referenced by the docs.
├── develop/                         — Docker Compose dev environment, driven by the Makefile.
│   ├── Dockerfile                   — Clones NetBox at NETBOX_VER and pip-installs this plugin -e.
│   ├── docker-compose.yml           — netbox, worker, postgres, redis.
│   ├── configuration.py             — NetBox config for the dev environment, incl. PLUGINS_CONFIG.
│   └── dev.env                      — Dev credentials (not secrets).
├── runtests.py                      — Standalone test runner; stubs netbox.plugins if absent.
├── setup.py                         — Packaging. install_requires, extras (test, docs), classifiers.
├── Makefile                         — Dev environment and release helpers.
├── CHANGELOG.md                     — The changelog. Canonical copy; docs/changelog.md includes it.
├── COMPATIBILITY.md                 — Plugin release to NetBox version matrix.
└── .github/workflows/
    ├── lint-tests.yaml              — docs build, package build, test matrix.
    └── pub-pypi.yml                 — Publishes to PyPI on a published GitHub release.
```

## Architecture

### The label pipeline

For each supported model, `template_content.py` defines a `PluginTemplateExtension` subclass declaring `models = ('dcim.device',)` and implementing `right_page()` or `left_page()`. Both delegate to `Create_PluginContent()`, and the chain is:

```
right_page()
  └─ Create_PluginContent()          — loops label designs 1..10, isolating failures
       └─ Create_SubPluginContent(n)  — renders one label

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netbox-community/netbox-qrcode](https://github.com/netbox-community/netbox-qrcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
