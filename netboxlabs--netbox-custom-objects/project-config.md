---
trigger: always_on
description: `netbox-custom-objects` is a NetBox plugin that lets users create custom object types at runtime without writing code. Each `CustomObjectType` definition generates a real Django model class backed by a real database table; instances of those models (custom objects) participate in NetBox's full feature set — tags, journals, change logging, search indexing, REST API, and more. It is owned by NetBox Labs and runs inside NetBox as a Django app (`netbox_custom_objects`, mounted at `/custom-objects/`)
---

# AGENTS.md — netbox-custom-objects

## Repository Overview

`netbox-custom-objects` is a NetBox plugin that lets users create custom object types at runtime without writing code. Each `CustomObjectType` definition generates a real Django model class backed by a real database table; instances of those models (custom objects) participate in NetBox's full feature set — tags, journals, change logging, search indexing, REST API, and more. It is owned by NetBox Labs and runs inside NetBox as a Django app (`netbox_custom_objects`, mounted at `/custom-objects/`). Requires PostgreSQL and NetBox 4.4.0+. The currently supported NetBox version range is in `COMPATIBILITY.md` (4.4.0 – 4.6.x at the time of writing).

## Tech Stack

- Python (defer to `pyproject.toml`; currently `>=3.10`)
- NetBox (host app — minimum and maximum versions are pinned in `netbox_custom_objects/__init__.py` `min_version` / `max_version`; `COMPATIBILITY.md` summarises the matrix)
- Django + Django REST Framework (NetBox's foundations)
- PostgreSQL (required — dynamic model tables are created directly via Django's schema editor)
- Redis (required — background reindex jobs use NetBox's job queue)
- Django's built-in test runner (`django.test.TestCase`-based, run via `manage.py test`)
- ruff for lint + format (config in `ruff.toml`)
- mkdocs + mkdocs-material for user-facing docs

Defer all version pins to `pyproject.toml` and `netbox_custom_objects/__init__.py`.

## Repository Map

```text
.
├── netbox_custom_objects/          — The Django app.
│   ├── __init__.py                 — PluginConfig (name, version, min/max NetBox, ready()).
│   ├── choices.py                  — ChoiceSet subclasses.
│   ├── constants.py                — APP_LABEL, RESERVED_FIELD_NAMES.
│   ├── dynamic_forms.py            — build_filterset_form_class() for HTMX object selector.
│   ├── field_types.py              — FieldType base + subclasses (one per supported field type).
│   ├── fields.py                   — Custom Django form/model field classes.
│   ├── filtersets.py               — get_filterset_class() for dynamically generated models.
│   ├── forms.py                    — Model forms for CustomObjectType and CustomObjectTypeField.
│   ├── jobs.py                     — ReindexCustomObjectTypeJob background job.
│   ├── models.py                   — CustomObject, CustomObjectType, CustomObjectTypeField.
│   ├── navigation.py               — Dynamic plugin menu construction.
│   ├── search.py                   — SearchIndex registrations for static models.
│   ├── tables.py                   — django-tables2 tables for list views.
│   ├── template_content.py         — PluginTemplateExtension registrations.
│   ├── urls.py                     — Web UI URL routing (80+ routes).
│   ├── utilities.py                — AppsProxy, generate_model(), get_viewname(), is_in_branch().
│   ├── views.py                    — All UI views.
│   ├── api/
│   │   ├── serializers.py          — get_serializer_class() + static serializers.
│   │   ├── urls.py                 — API URL routing.
│   │   └── views.py                — Dynamic ViewSet generation + LinkedObjectsView.
│   ├── migrations/                 — Django schema migrations (0001–0004).
│   ├── templates/netbox_custom_objects/
│   │   ├── buttons/
│   │   ├── htmx/
│   │   └── inc/
│   ├── templatetags/
│   │   ├── custom_object_buttons.py
│   │   └── custom_object_utils.py
│   └── tests/
│       ├── base.py                         — Shared test utilities and base cases.
│       ├── test_api.py                     — REST API endpoints (CRUD, linked objects).
│       ├── test_deletion.py                — Cascade deletion behaviour.
│       ├── test_field_types.py             — FieldType subclass behaviour.
│       ├── test_filtersets.py              — Filterset functionality.
│       ├── test_models.py                  — CustomObjectType and field model logic.
│       ├── test_navigation.py              — Dynamic navigation menu.
│       ├── test_schema_operations.py       — Schema creation/deletion/alteration.
│       └── test_views.py                   — Web views.
├── docs/
│   ├── api.md
│   ├── branching.md
│   ├── changelog.md
│   ├── configuration.md
│   └── index.md
├── testing/
│   └── configuration.py            — NetBox config used by the test workflow.
├── .github/workflows/
│   ├── claude.yaml                 — Claude Code automation hook.
│   ├── lint-tests.yaml             — Lint + test CI (runs on every push/PR).
│   └── release.yaml                — PyPI publish on GitHub release.
├── AGENTS.md                       — This file.
├── CLAUDE.md                       — Shim that pulls in this file.
├── COMPATIBILITY.md                — Plugin → NetBox version matrix.
├── pyproject.toml                  — Plugin metadata + dependencies.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netboxlabs/netbox-custom-objects](https://github.com/netboxlabs/netbox-custom-objects) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
