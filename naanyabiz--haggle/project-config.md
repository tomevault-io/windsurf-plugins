---
trigger: always_on
description: generates a PKCE verifier+challenge, builds an `/authorize` URL, and shows
---

# AGENTS.md — Haggle Integration Guide

> **One-liner**: `haggle` is a Home Assistant custom integration that pulls AGL Australia
> smart-meter interval data from AGL's undocumented REST API and feeds it into HA's
> Energy dashboard via `import_statistics()`.

This file is the canonical documentation for both human contributors and AI agents.
`CLAUDE.md` is a symlink to this file.

---

## Dev Loop

```bash
# Install deps (once, or after pyproject.toml changes)
uv sync

# Run tests
uv run pytest

# Lint + format
uv run ruff check --fix custom_components/ tests/
uv run ruff format custom_components/ tests/

# Type-check
uv run mypy custom_components/haggle

# Validate manifest
python scripts/validate_manifest.py custom_components/haggle/manifest.json

# Run all pre-commit hooks
uv run pre-commit run --all-files

# Hassfest — easiest via CI (push a branch + open PR)
# Or use the dedicated image locally:
docker run --rm \
  -v "$(pwd)/custom_components:/github/workspace/custom_components:ro" \
  ghcr.io/home-assistant/hassfest \
  --integration-path /github/workspace/custom_components/haggle
```

---

## Repo Map

```
custom_components/haggle/
├── __init__.py          # async_setup_entry / async_unload_entry / async_remove_entry + HaggleRuntimeData
├── manifest.json        # HACS/HA metadata; hassfest validates this
├── const.py             # all constants — DOMAIN, API hosts, config-entry keys, data keys
├── config_flow.py       # PKCE authorize URL → user pastes callback → exchange → select_contract
├── coordinator.py       # HaggleCoordinator: 30-day backfill (throttled, 429-aware) + incremental statistics import (aggregate + per-tariff ToU series)
├── sensor.py            # 9 SensorEntityDescription entries (3 conditional ToU rate sensors); HaggleEnergySensor
├── agl/
│   ├── __init__.py
│   ├── client.py        # AglAuth (JWT expiry + token rotation) + AglClient (HTTP methods)
│   ├── models.py        # TokenSet, Contract, IntervalReading, DailyReading, BillPeriod, PlanRates
│   ├── parser.py        # JSON → typed dataclasses (filters type=none intervals)
│   └── pinning.py       # SPKI extraction helper for Trust-On-First-Use TLS pinning
├── strings.json         # translatable config-flow strings
└── translations/en.json # English strings (must mirror strings.json)

tests/
├── conftest.py                      # _auto_enable_custom_integrations fixture
├── fixtures/
│   ├── hourly_response.json         # 30-min interval data (Current/Hourly)
│   ├── overview_response.json       # /v3/overview with accounts + contracts
│   ├── plan_response.json           # /v2/plan/energy with gstInclusiveRates (flat rate)
│   ├── tou_plan_response.json       # Time-of-Use plan — per-band gstInclusiveRates
│   ├── tou_hourly_response.json     # mixed peak/offpeak/shoulder/normal intervals
│   └── bill_period_response.json    # usage summary
├── test_init.py                     # setup/unload smoke tests
├── test_config_flow.py              # PKCE step navigation (user → exchange → select_contract)
├── test_agl_client.py               # AglAuth token rotation + AglClient HTTP methods + pin-check wiring
├── test_const.py                    # base64 sanity-check on AGL_AUTH0_CLIENT
├── test_parser.py                   # parse_interval_readings, parse_overview, parse_plan, ToU rate mapping, _safe_float
├── test_pinning.py                  # SPKI extraction + host-name guards
├── test_coordinator_statistics.py   # backfill, incremental resume, idempotency, ToU per-tariff series, numeric guards
└── test_sensor.py                   # sensor descriptions + conditional ToU rate-sensor registration

scripts/
├── wt                   # bash worktree helper (new / list / rm)
└── validate_manifest.py # used by the validate-manifest Claude hook

.claude/
├── settings.json        # committed hooks config
├── agents/              # 8 subagent definitions (5 domain + 3 review)
└── commands/            # 5 slash commands (new-entity, wt, release, hassfest, pr)

.github/
├── workflows/
│   ├── ci.yml           # ruff + mypy + pytest matrix (Python 3.13)
│   ├── hacs.yml         # HACS validation
│   ├── hassfest.yml     # Home Assistant integration manifest validation
│   ├── release.yml      # tag-triggered GitHub Release + build-provenance attestation
│   └── codeql.yml       # weekly + per-PR CodeQL Python scan
├── CODEOWNERS           # @naanyabiz owns everything
└── dependabot.yml       # weekly pip + github-actions updates

# Repo-root posture files
SECURITY.md              # disclosure path + threat-model summary
CONTRIBUTING.md          # dev loop + commit conventions + PR checklist
CODE_OF_CONDUCT.md       # Contributor Covenant 2.1
```

---

## Documentation Checklist — Required on Every PR

Every PR that ships code (not pure CI/tooling fixes) MUST include updates to
all of the following before it can be merged. The `/pr` command enforces this.

| Artifact | What to update | Where |
|---|---|---|
| `CHANGELOG.md` | Add bullet(s) under `## [Unreleased]` for every user-visible capability added, changed, or fixed | repo root |
| `AGENTS.md` — Repo Map | Add any new files; update descriptions if a file's role changed | this file |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NaanyaBiz/haggle](https://github.com/NaanyaBiz/haggle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
