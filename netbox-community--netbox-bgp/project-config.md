---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`netbox-bgp` is a [NetBox](https://github.com/netbox-community/netbox) plugin that adds BGP-related objects (Sessions, Peer Groups, Communities, Community Lists, Routing Policies + Rules, Prefix Lists + Rules, AS Path Lists + Rules). The plugin is distributed on PyPI as `netbox-bgp` and installs as a Django app named `netbox_bgp`.

The NetBox ⇄ plugin version pairing is strict (declared in `netbox_bgp/__init__.py` via `min_version`/`max_version` and tabulated in `COMPATIBILITY.md`, linked from `README.md`). The current release (`0.19.0`, version in `netbox_bgp/version.py`) targets NetBox 4.6.x (`min_version = '4.6.0'`, `max_version = '4.6.99'`); development happens on the `develop` branch. Changing the plugin's NetBox version range almost always requires migrations and import changes against NetBox internals — add a new row to `COMPATIBILITY.md` when you do.

## Development workflow

The `develop/` directory contains a full Docker Compose stack (NetBox + worker + Postgres + Redis). The plugin source is bind-mounted into the NetBox container, so edits on the host take effect after a container restart (or `runserver` autoreload). The `Makefile` wraps everything — prefer it over raw `docker compose`.

Common commands (all run via `make`):

- `make cbuild` — build the dev image. Pass `NETBOX_VER=vX.Y.Z PYTHON_VER=3.12` to target a different NetBox tag.
- `make debug` / `make start` / `make stop` — run the stack in foreground / detached / stop.
- `make destroy` — stop the stack **and drop the Postgres volume** (`netbox_bgp_pgdata_netbox_bgp`). Use when migrations get wedged.
- `make migrations` — generate Django migrations for the plugin (writes into `netbox_bgp/migrations/`). Required after any `models.py` change.
- `make test` — run `python manage.py test netbox_bgp` inside the container. Tests live in `netbox_bgp/tests/` (`test_api.py`, `test_filtersets.py`, `test_forms.py`, `test_models.py`, `test_search.py`, `test_views.py`). Run a single test with: `docker compose -f develop/docker-compose.yml -p netbox_bgp run netbox python manage.py test netbox_bgp.tests.test_api.SomeTestCase.test_method`.
- `make nbshell` / `make shell` — NetBox shell / Django shell.
- `make adduser` — create a superuser.
- `make pbuild` / `make pypipub` — build sdist/wheel / upload to PyPI.
- `make relpatch` — bump patch version on a release branch (requires clean working tree; runs `pysemver bump patch` against `netbox_bgp/version.py`).

The version string lives in a single place: `netbox_bgp/version.py` (read by both `setup.py` and `PluginConfig`).

## Architecture

This is a standard NetBox plugin — `netbox_bgp/__init__.py` exposes a `PluginConfig` subclass as `config`, which NetBox discovers when the plugin is listed in `PLUGINS`. Everything else follows NetBox's plugin conventions; when adding features, mirror the patterns used by existing models rather than inventing new ones.

### Models (`netbox_bgp/models.py`)

All models inherit from `netbox.models.NetBoxModel` (giving them change-logging, tags, custom fields, journaling). Two shapes are used heavily:

- **List / Rule pairs**: `PrefixList` ↔ `PrefixListRule`, `CommunityList` ↔ `CommunityListRule`, `ASPathList` ↔ `ASPathListRule`, `RoutingPolicy` ↔ `RoutingPolicyRule`. Rules are ordered by `(parent, index)` and `on_delete=CASCADE` from the parent.
- **`BGPBase` abstract**: provides `site`, `tenant`, `status`, `role`, `description`, `comments`. Currently only `Community` inherits it; `BGPSession` is a standalone model with its own richer FK set (device/vm, local/remote IP + AS, peer group, import/export policies, prefix lists in/out, `remote_as_macro`, `extra_attributes`). `BGPPeerGroup` also carries session-level config fields (`local_as`, `remote_as`, `prefix_list_in`, `prefix_list_out`, `extra_attributes`) that serve as group-wide defaults.

`BGPSession.label` falls back to `f'{remote_address}:{remote_as}'` when `name` is unset — don't introduce code that assumes `name` is always populated. `BGPSession.Meta.unique_together` covers both device- and VM-scoped tuples; a session must have exactly one of device/VM in practice (the `clean()` enforcement is currently commented out — see `models.py:489`).

`PrefixListRule` has dual prefix fields: `prefix` (FK to `ipam.Prefix`) and `prefix_custom` (`IPNetworkField`). Exactly one must be set; this is enforced in `clean()`. The `network` property returns whichever is populated.

### Surface area (one module per concern)

- `api/` — DRF viewsets/serializers mounted via `NetBoxRouter`. Note: both `bgpsession`/`session` and `bgppeergroup`/`peer-group` are registered as aliases for backwards compatibility (`api/urls.py`). When adding endpoints, register both spellings only if you're preserving an alias; otherwise pick one.
- `graphql/` — strawberry-django schema, types, filters, enums. The top-level `NetBoxBGPQuery` type exposes `netbox_bgp_*` fields; field names are load-bearing for GraphQL clients.
- `filtersets.py` / `forms.py` / `tables.py` — FilterSets (django-filter), edit/filter/bulk forms, and django-tables2 tables.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netbox-community/netbox-bgp](https://github.com/netbox-community/netbox-bgp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
