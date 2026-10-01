---
trigger: always_on
description: Guidance for Claude Code sessions in this repository.
---

# CLAUDE.md

Guidance for Claude Code sessions in this repository.

## What this is

`retry` is a Home Assistant custom integration (distributed via HACS, domain `retry`) that adds two actions:

- `retry.action`: the engine. It calls one inner action (`action: light.turn_on`, …) and retries it on failure with a templated backoff. Optionally it validates the result (`expected_state`, `validation`) and runs `on_error` after the final failure.
- `retry.actions`: the UI-friendly wrapper. It walks a script `sequence`, rewrites every `call_service` step into a `retry.action` call carrying the shared retry parameters, and runs the result as an ad-hoc `script.Script`.

README.md is the user-facing spec for every parameter. Keep it in sync with the code.

## Layout

| Path | Role |
|------|------|
| `custom_components/retry/__init__.py` | Almost all logic: schemas, `RetryParams` (parse/validate, target resolution), `RetryAction` (per-entity retry loop), `_wrap_actions` (retry.actions rewriting), service registration in `async_setup` |
| `custom_components/retry/config_flow.py` | Single-instance config flow + options flow (`disable_initial_check`, `disable_repair`) |
| `custom_components/retry/const.py` | Constants / parameter names |
| `custom_components/retry/diagnostics.py` | Returns `{}` |
| `services.yaml`, `strings.json`, `translations/*.json`, `icons.json` | Action UI metadata. `translations/en.json` mirrors `strings.json` with the `[%key:…%]` references expanded |
| `tests/test_init.py` | Nearly all behavior tests (~90 cases) |
| `config/configuration.yaml` | Dev HA instance config for `scripts/develop` |

## Commands

```bash
scripts/setup            # install pytest-homeassistant-custom-component (pre-releases allowed), ruff, mypy, prek; install hooks
scripts/lint             # ruff format + ruff check --fix + mypy --strict custom_components/retry
scripts/lint --no-fix    # what CI runs
pytest                   # pytest.ini adds --cov ... --cov-fail-under=100 (100% line coverage is REQUIRED)
pytest tests/test_init.py -k retry_id -o addopts=""   # quick targeted run without the coverage gate
scripts/develop          # run a dev HA on :8123 with ./config
```

The pre-commit hooks (`prek.toml`) run lint and the full pytest suite.

## Architecture notes (non-obvious)

- **Services are registered in `async_setup`**, not per entry. Each call looks up the single config entry (`get_config_entry()`) and fails with `ServiceValidationError` if the entry is missing or not loaded. When no entry exists, `async_setup` starts an import flow (the `retry:` YAML key or the UI both end up with one entry).
- **Callers render templates before the handler runs.** When `retry.action`/`retry.actions` is called from an automation or script, HA's `template_complex` / `render_complex` renders every `{{ }}` string nested anywhere in `data` *before* the service handler runs. That is why `backoff` and `validation` use the special `[[ … ]]` / `[% … %]` / `[# … #]` syntax, converted by `_fix_template_tokens`. The catch: `on_error` templates and templates inside a `retry.actions` `sequence` are rendered by the caller, with the caller's variables. `{% raw %}` defers them.
- **`retry.actions` keeps `on_error` raw.** `_script_schema_validate_only` validates it but passes the original structure on, so it isn't double-converted into `Template` objects.
- **Target resolution** (`RetryParams._entity_ids`): a string `entity_id` is first normalized, best-effort, with `cv.comp_entity_ids`. That way comma-separated/padded ids and any-case `ALL`/`NONE` resolve like their canonical forms. Strings entity services would reject (e.g. `""`) are used as-is. Only strings have non-canonical forms; list items are lowercased by the target helper. It then uses `homeassistant.helpers.target.async_extract_referenced_entity_ids` (HA already expands old-style `group.*` and `GenericGroup` entities). `_expand_group` additionally expands group-*platform* entities (light/switch/… groups) through their `entity_id` attribute. Indirect references (area/device/floor/label) are filtered to the action's domain/integration, except `homeassistant.*`. Each resolved entity gets its own `RetryAction` loop, run with `asyncio.gather`.
- **Availability and state checks use `Entity` objects** from `hass.data[DATA_INSTANCES]` (`_get_entity`), not the state machine. This matches HA's own "skip unavailable entities" logic in `entity_service_call`.
- **`retry_id`** (default: entity_id, else the action name) lives in the module-global `_running_retries: {retry_id: (context.id, count)}`. A newer call with a different `Context` takes ownership. The older loop notices at its next attempt boundary and raises `IntegrationError`. Loops sharing a context (e.g. several entities of one call) share the counter.
- **Repairs** are created on the final failure. The issue id is `str(RetryAction)`, which includes the inner action data. They're non-persistent and never deleted on a later success.
- `TargetSelection` is imported with a fallback to `TargetSelectorData`. Older HA (e.g. 2025.8) only has the latter, which is deprecated in current HA and breaks in 2026.12.

## Testing conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amitfin/retry](https://github.com/amitfin/retry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
