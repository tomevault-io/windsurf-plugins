---
trigger: always_on
description: A Home Assistant custom integration for Jev, TypeSafe's decision model. A user writes
---

# HA-Jev

A Home Assistant custom integration for Jev, TypeSafe's decision model. A user writes
a question about their home in plain language and gets a typed answer back, as a
sensor, an action response, a conversation agent reply or an AI Task result. It ships
through HACS. The domain is `jev` and all runtime code is in `custom_components/jev`.

## What we do not trade away

### Every request costs the user money

Each evaluation is a paid API call, billed per input token. A change that sends a
request needs a schedule or debounce that a flapping entity cannot defeat, and a place
in the daily budget. A scheduled or automated request that would not fit the budget
is refused before it is sent. Tests replace the client, so the suite spends nothing, and it has to stay that
way.

### Platinum on the quality scale

`quality_scale.yaml` records every rule, and each `done` names the test or measurement
behind it. When a change moves a number quoted there (tests, modules, translation
keys, coverage), update the number in the same commit. When a change breaks a rule,
fix the code.

### Answers you can act on

An entity with no answer for its exact question is unavailable, because a missing
answer is missing data and not a value (`entity.py`). A choice or score answer carries
its confidence as an attribute. A noul carries none, because the probability is the
whole answer.

## Glossary

- **question** is one subentry and one sensor.
- **context** is a group of questions that share one state and one API call. Each
  context has its own coordinator.
- **state** is the text Jev judges, built from targets or a template in
  `statebuilder.py`.
- **noul**, **choice** and **score** are the three question types. A noul is a
  probability between 0 and 1.
- **budget** is the daily input-token limit of one config entry.
- **payload** is the request body. `payload.py` mirrors jevclient's wire format so the
  size can be measured before the request goes out.

## The three ways to hurt yourself

1. **Running `tests/live`.** It drives a real Home Assistant with a real API key and
   spends real tokens. It exists only on the maintainer's machine, and `.gitignore`
   keeps it out of the repository. Run it only when the maintainer asks. `pytest.ini`
   keeps it out of collection with `norecursedirs`, and that line stays.
2. **Publishing the maintainer's home.** The repository is public. In code, tests,
   docs and examples, use addresses from `192.0.2.0/24` and generic entity ids such as
   `sensor.washing_machine_power`. Real hostnames, LAN addresses, room names and keys
   stay out.
3. **Editing one language.** User-visible text lives in `strings.json` and in 13 files
   under `translations/`. Every key and every placeholder must exist in all of them.

## Hit every surface

The common defect here is a change that works on the path you tested and is missing
on the others. Before calling a change done, walk this list and say which entries
applied:

- **Request paths.** A question reaches Jev through `coordinator.py` (subentries and
  contexts on a schedule), `services.py` (`jev.noul`, `jev.choice`, `jev.score`,
  `jev.ask`), `conversation.py`, `ai_task.py` and the preview in `subentry.py`. A change
  to how a request is built, budgeted or reported needs a decision for each one. Today
  the coordinator and AI Task refuse a call that would not fit before sending it, the
  conversation agent and the preview stop once the budget is spent, and the actions
  record what they spend without refusing.
- **Flows.** The config flow has user, reauth and reconfigure steps, plus options. A
  new field goes into every step where it applies.
- **Entities.** A new entity gets a `translation_key`, an icon in `icons.json` and an
  entity category. A diagnostic or heavy entity ships disabled by default.
- **Docs.** The README, the page in `site-docs/`, and a troubleshooting row when the
  change adds a failure a user can see.

## Home Assistant rules the suite enforces

- Raise the `HomeAssistantError` family with `translation_domain`, `translation_key`
  and `translation_placeholders`. `tests/test_translations.py` walks every module and
  fails on a bare string.
- `strings.json` cannot contain a URL. hassfest refuses it. Pass the URL through the
  form's `description_placeholders`.
- `requirements-test.txt` pins exactly what Home Assistant pins for the components this
  integration imports. `tests/test_manifest.py` fails when they drift.
- Enabling an entity in the registry schedules a reload 30 s later. In tests, use
  `enable_entity` in `tests/test_init.py` and do not reload by hand.

## Verifying

The full gate takes under half a minute, so run all of it before calling work done:

```bash
.venv/bin/ruff check ./custom_components ./tests
.venv/bin/ruff format --check ./custom_components ./tests
.venv/bin/mypy --strict --ignore-missing-imports custom_components/jev
.venv/bin/pytest
.venv/bin/mkdocs build --strict
docker run --rm -v "$PWD":/github/workspace ghcr.io/home-assistant/hassfest
```

Always use `.venv/bin/python` and the tools in `.venv/bin`. `config_flow.py` stays at
100% coverage, which is a Bronze rule. New behaviour ships with a test of what a user

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AboveColin/HA-Jev](https://github.com/AboveColin/HA-Jev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
