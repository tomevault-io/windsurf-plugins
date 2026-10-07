---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`netbox-contract` is a NetBox plugin (Django app `netbox_contract`) managing contracts, contract lines, units,
invoices, invoice lines, accounting dimensions, service providers and contract assignments to NetBox objects.
Minimum NetBox 4.6 (`min_version` in `netbox_contract/__init__.py`), Python 3.12+. Project rules are in
`.specify/memory/constitution.md` (Principles I-VII); it overrides other practice documents when they conflict.

## Environment and commands

The plugin is developed next to a NetBox checkout and installed in NetBox's virtualenv in editable mode
(`pip install -e .`). In the dev container the layout is `/workspaces/netbox/netbox` (NetBox, venv in `venv/`) and
`/workspaces/netbox/netbox-contract` (this repo). Management commands run from the NetBox checkout.

```bash
# Lint (also run by the pre-commit hook on netbox_contract/; never bypass the hook)
ruff check

# Tests: run from the NetBox checkout. The dev configuration has DEBUG on, which the debug toolbar
# refuses under tests, so use the test configuration (a copy of testing/configuration.py for the container).
cd ../netbox
NETBOX_CONFIGURATION=netbox.configuration_testing venv/bin/python netbox/manage.py test netbox_contract.tests --keepdb
# One module / class / test
NETBOX_CONFIGURATION=netbox.configuration_testing venv/bin/python netbox/manage.py test \
    netbox_contract.tests.test_generation.GenerationRulesTestCase.test_credit_line --keepdb

# Query-count baselines (netbox_contract/tests/query_counts.json): list view tests fail when a list view's
# query count changes. Re-record serially (no --parallel) and only with a stated reason (Constitution II).
UPDATE_QUERY_COUNTS=1 NETBOX_CONFIGURATION=netbox.configuration_testing venv/bin/python netbox/manage.py test \
    netbox_contract.tests.test_views netbox_contract.tests.test_api --keepdb

# Migrations: makemigrations is refused unless the NetBox configuration sets DEVELOPER = True.
venv/bin/python netbox/manage.py makemigrations netbox_contract --check --dry-run
venv/bin/python netbox/manage.py migrate netbox_contract

# Re-run the legacy data conversion (idempotent, prints a report)
venv/bin/python netbox/manage.py convert_contract_lines
```

CI (`.github/workflows/lint-tests.yaml`) links `testing/configuration.py` as NetBox's configuration and runs the
suite on a pinned NetBox tag (currently `v4.6.10`) for Python 3.12-3.14.

## Architecture

**Standard NetBox plugin stack per model**: `models.py`, `forms.py` (edit, filter, CSV import, bulk edit),
`filtersets.py`, `tables.py`, `views.py`, `urls.py`, `api/` (serializers, viewsets, router), `graphql/` (filters,
types, schema; registered by `PluginConfig.graphql_schema`), `search.py`, `navigation.py`, templates under
`templates/netbox_contract/` (never at the root of `templates/`). Every model is a `NetBoxModel` with the full
stack; `ContractType` is an `OrganizationalModel` (its slug is derived from the name when empty, `text.unique_slug`)
and `ServiceProvider` a `PrimaryModel`, with the matching core form, filterset, table, serializer and GraphQL bases.
Every view is registered with `register_model_view` (list, add, import, bulk edit/delete with `detail=False`);
`urls.py` only includes `get_model_urls` per model plus the non-model `invoice_lines_preview`, so NetBox adds
changelog and journal and other code can attach tabs and actions (e.g. `contractline_amend`). Route names are pinned
by `tests/test_conventions.py`. Filtersets are registered with `@register_filterset` (lookup modifiers on filter
forms). Model, bulk-edit and filter forms declare `fieldsets`: a field left out of every section is not rendered,
and `prune_fieldsets` removes fields deleted (deprecated) or hidden by the settings.
- New-object pre-fill (invoice, invoice line) uses `form_with_defaults` in `views.py` and then core
  `ObjectEditView.get()`, so quick add and HTMX partials work; values in the page address win over the pre-fill.
- Detail pages are declared with `layout = SimpleLayout(...)` on the `ObjectView`s; the plugin's panels are in
  `panels.py` (`SettingsAttributesPanel` applies `hidden_contract_fields` / `hidden_invoice_fields` at render time,
  deprecated panels follow `show_deprecated_fields`, messages are `TemplatePanel` fragments under
  `templates/netbox_contract/panels/`). Related tables are `ObjectsTablePanel`s that load the list views over HTMX
  (tests read them with `tests.helpers.contract_page_with_lines` or the panel's `hx-get` URL). Only the contract,
  contract line, invoice and invoice line pages keep a template, for their breadcrumbs.
- Amending a contract line needs the `amend` permission action (`ContractLine.Meta.permissions`,
  `netbox_contract.amend_contractline`), checked per line: `object_actions.AmendContractLine` (page),
  `tables.ContractLineActionsColumn` (tables; also hides Delete on locked lines), `can_amend` template filter (edit
  page), `ContractLineAmendView` and the REST `amend` action, which overrides NetBox's POST → "add" mapping
  (`get_permissions()`). The contract line list reads lock and successor state from

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mlebreuil/netbox-contract](https://github.com/mlebreuil/netbox-contract) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
