---
trigger: always_on
description: `netbox-branching` is a NetBox plugin that adds git-like branching to the network source-of-truth platform. Each branch is an isolated PostgreSQL schema copy of the database; users make changes within a branch and merge back to the main schema. It is owned by NetBox Labs and runs inside NetBox as a Django app (`netbox_branching`, mounted at `/branching/`). Requires PostgreSQL (schema isolation), Redis (background jobs), and NetBox 4.4.1+. The currently supported NetBox version range is in `COMPA
---

# AGENTS.md — netbox-branching

## Repository Overview

`netbox-branching` is a NetBox plugin that adds git-like branching to the network source-of-truth platform. Each branch is an isolated PostgreSQL schema copy of the database; users make changes within a branch and merge back to the main schema. It is owned by NetBox Labs and runs inside NetBox as a Django app (`netbox_branching`, mounted at `/branching/`). Requires PostgreSQL (schema isolation), Redis (background jobs), and NetBox 4.4.1+. The currently supported NetBox version range is in `COMPATIBILITY.md` (4.4.1 – 4.6.x at the time of writing).

## Tech Stack

- Python (defer to `pyproject.toml`; currently `>=3.10`)
- NetBox (host app — minimum and maximum versions are pinned in `netbox_branching/__init__.py` `min_version` / `max_version`; `COMPATIBILITY.md` summarises the matrix)
- Django + Django REST Framework (NetBox's foundations)
- PostgreSQL (required — branch isolation depends on schema-level separation)
- Redis (required — background jobs use NetBox's job queue)
- Django's built-in test runner (`django.test.TestCase`-based, run via `manage.py test`)
- ruff for lint + format (config in `ruff.toml`)
- mkdocs + mkdocs-material for user-facing docs

Defer all version pins to `pyproject.toml` and `netbox_branching/__init__.py`.

## Repository Map

```text
.
├── netbox_branching/          — The Django app.
│   ├── __init__.py            — PluginConfig (name, version, min/max NetBox); validates settings in ready().
│   ├── choices.py             — ChoiceSet subclasses (branch status, merge strategy, event type).
│   ├── constants.py           — BRANCH_ACTIONS list.
│   ├── contextvars.py         — active_branch ContextVar (propagates through async automatically).
│   ├── database.py            — BranchAwareRouter — custom Django database router.
│   ├── error_report.py        — Error reporting helpers.
│   ├── events.py              — BranchEvent creation and management.
│   ├── filtersets.py          — BranchFilterSet, BranchEventFilterSet, ChangeDiffFilterSet.
│   ├── jobs.py                — AsyncJob subclasses: Provision, Sync, Merge, Revert, Migrate.
│   ├── middleware.py          — BranchMiddleware: per-request branch context activation.
│   ├── navigation.py          — Plugin menu definition.
│   ├── object_actions.py      — ObjectAction subclasses for branch operations.
│   ├── search.py              — SearchIndex registrations.
│   ├── signal_receivers.py    — Django signal handlers (post_save, pre_delete, etc.).
│   ├── signals.py             — Pre/post branch operation signals (pre_sync, post_merge, etc.).
│   ├── template_content.py    — PluginTemplateExtension registrations.
│   ├── urls.py                — Top-level URL routing.
│   ├── utilities.py           — DynamicSchemaDict, branch activation helpers, change replay.
│   ├── views.py               — All UI views.
│   ├── webhook_callbacks.py   — Webhook/event rule integration.
│   ├── api/
│   │   ├── serializers.py
│   │   ├── urls.py            — NetBoxRouter registrations.
│   │   └── views.py           — BranchViewSet (with sync/merge/revert/migrate actions).
│   ├── forms/
│   │   ├── bulk_edit.py
│   │   ├── bulk_import.py
│   │   ├── filtersets.py      — Filter forms for list views.
│   │   ├── misc.py
│   │   └── model_forms.py
│   ├── merge_strategies/
│   │   ├── strategy.py        — Abstract MergeStrategy base class.
│   │   ├── iterative.py       — IterativeMergeStrategy (default).
│   │   └── squash.py          — SquashMergeStrategy.
│   ├── migrations/            — Django schema migrations (0001–0008).
│   ├── models/
│   │   ├── __init__.py        — Star-imports every submodule.
│   │   ├── branches.py        — Branch, BranchEvent.
│   │   └── changes.py         — ObjectChange (proxy), ChangeDiff, AppliedChange.
│   ├── tables/
│   │   ├── columns.py
│   │   └── tables.py
│   ├── templatetags/
│   │   ├── branch_buttons.py
│   │   └── branch_filters.py
│   ├── templates/netbox_branching/
│   │   ├── buttons/
│   │   └── inc/
│   └── tests/
│       ├── utils.py                    — Shared test utilities.
│       ├── test_api.py
│       ├── test_branches.py
│       ├── test_changediff.py
│       ├── test_config.py
│       ├── test_connection_lifecycle.py
│       ├── test_events.py
│       ├── test_filtersets.py
│       ├── test_iterative_merge.py
│       ├── test_query.py
│       ├── test_related_models.py
│       ├── test_request.py
│       ├── test_squash_merge.py
│       ├── test_sync.py
│       └── test_views.py
├── docs/                      — mkdocs site.
│   ├── models/                — Per-model documentation.
│   └── using-branches/        — User guides.
├── testing/
│   └── configuration.py       — NetBox config used by the test workflow.
├── .github/workflows/         — lint-tests.yaml, release.yaml, claude.yaml.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netboxlabs/netbox-branching](https://github.com/netboxlabs/netbox-branching) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
