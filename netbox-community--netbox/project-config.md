---
trigger: always_on
description: NetBox is an extensible open-source network source-of-truth application powering network automation. It manages network infrastructure data including data center infrastructure (DCIM), IP address management (IPAM), circuits, virtualization, wireless, VPNs, and more. It supports a plugin ecosystem and exposes both a REST API and GraphQL API.
---

# NetBox

## Repository Overview

NetBox is an extensible open-source network source-of-truth application powering network automation. It manages network infrastructure data including data center infrastructure (DCIM), IP address management (IPAM), circuits, virtualization, wireless, VPNs, and more. It supports a plugin ecosystem and exposes both a REST API and GraphQL API.

NetBox is the core product maintained by NetBox Labs. The current version is 4.6 (Python 3.12+, Django 6.x).

## Tech Stack

- Python 3.12+ / Django 6.x / Django REST Framework 3.x
- PostgreSQL (required), Redis (required for caching/queuing)
- GraphQL via Strawberry, background jobs via django-rq
- django-tables2 for list views, django-filter for filtering
- drf-spectacular for OpenAPI/Swagger schema generation
- Docs: MkDocs with mkdocs-material theme (in `docs/`)
- Ruff for lint (config in `pyproject.toml`)

## Repository Map

```text
.
├── netbox/                    — Django project root (run manage.py from here)
│   ├── manage.py
│   ├── netbox/                — Core settings, URLs, WSGI, plugin infrastructure
│   │   ├── settings.py        — Main Django settings
│   │   ├── configuration.py   — Instance configuration (gitignored)
│   │   ├── configuration_example.py — Configuration template
│   │   ├── configuration_testing.py — Test configuration
│   │   ├── urls.py            — Root URL routing
│   │   ├── wsgi.py            — WSGI entrypoint
│   │   ├── api/               — Core REST API infrastructure
│   │   ├── graphql/           — Core GraphQL schema
│   │   ├── models/            — Core model infrastructure (features, mixins)
│   │   ├── navigation/        — Navigation menu system
│   │   ├── plugins/           — Plugin system infrastructure
│   │   ├── registry.py        — Object registry
│   │   ├── search/            — Full-text search implementation
│   │   ├── ui/                — UI utilities
│   │   └── tests/             — Core framework tests
│   ├── account/               — User account management
│   ├── circuits/              — Circuit and provider management
│   ├── core/                  — Core data management (data sources, jobs)
│   ├── dcim/                  — Data center infrastructure (devices, racks, cables, etc.)
│   ├── extras/                — Cross-cutting features (custom fields, tags, webhooks, scripts)
│   ├── ipam/                  — IP address management (prefixes, addresses, VLANs, etc.)
│   ├── tenancy/               — Tenancy and organization
│   ├── users/                 — User management and tokens
│   ├── utilities/             — Shared utilities (no models)
│   ├── virtualization/        — Virtual machines and clusters
│   ├── vpn/                   — VPN tunnels and configurations
│   ├── wireless/              — Wireless LANs and links
│   ├── templates/             — Django templates (per-app subdirectories)
│   ├── static/                — Compiled static assets
│   ├── project-static/        — Source static assets
│   ├── media/                 — User-uploaded media
│   └── translations/          — i18n translation files
├── docs/                      — MkDocs documentation source
│   ├── administration/
│   ├── configuration/
│   ├── customization/
│   ├── development/           — Contributing guide, code style
│   ├── features/
│   ├── getting-started/
│   ├── installation/
│   ├── integrations/
│   ├── models/                — Per-model documentation (by app)
│   ├── plugins/
│   ├── reference/
│   └── release-notes/
├── scripts/                   — Database management and verification scripts
├── contrib/                   — Example configs (systemd, nginx, generated schemas)
├── pyproject.toml             — Project metadata, ruff config
├── requirements.txt           — Python dependencies
└── mkdocs.yml                 — Docs site configuration
```

## Architecture

### App Structure

Each Django app (account, circuits, core, dcim, extras, ipam, tenancy, users, virtualization, vpn, wireless) follows a standard layout:

```text
<app>/
├── __init__.py
├── models/          — Database models (or models.py for smaller apps)
├── migrations/      — Database migrations
├── api/
│   ├── serializers.py
│   ├── views.py     — DRF viewsets
│   └── urls.py      — NetBoxRouter registrations
├── forms/           — Django forms (model forms, filter forms, bulk edit, etc.)
├── tables/          — django-tables2 table definitions
├── graphql/
│   └── types.py     — Strawberry GraphQL types
├── filtersets.py    — django-filter FilterSets
├── choices.py       — ChoiceSet subclasses
├── views.py         — UI views (registered with register_model_view())
├── urls.py          — URL routing
├── search.py        — SearchIndex registrations
├── signals.py       — Django signal definitions (where applicable)
└── tests/
    ├── test_api.py
    ├── test_filtersets.py
    ├── test_models.py
    ├── test_views.py
    └── test_forms.py
```

### Views


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netbox-community/netbox](https://github.com/netbox-community/netbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
