---
trigger: always_on
description: > For AI agents. Read the section relevant to your task — you don't need to read everything every time.
---

# ha-nanit — AGENTS.md

> For AI agents. Read the section relevant to your task — you don't need to read everything every time.
> Use the [Context Router](#context-router) to find which sections apply.
> Human contributors: see [CONTRIBUTING.md](CONTRIBUTING.md).

## Context Router

| If your task involves…              | Read sections                                      |
|--------------------------------------|----------------------------------------------------|
| Any code change                      | [Code Standards](#code-standards), [Git Workflow](#git-workflow), [Guardrails](#guardrails) |
| Integration code (`custom_components/`) | Above + [Architecture](#architecture), [HA Integration Patterns](#ha-integration-patterns) |
| Client library (`packages/aionanit/`)  | Above + [Architecture](#architecture), [aionanit Patterns](#aionanit-patterns) |
| Connection/WebSocket work            | Above + [docs/CONNECTION_RELIABILITY.md](docs/CONNECTION_RELIABILITY.md) |
| Security review                      | [Security](#security), [docs/SECURITY_AUDIT_CHECKLIST.md](docs/SECURITY_AUDIT_CHECKLIST.md) |
| PR review                            | [Git Workflow](#git-workflow), [Security](#security), [Guardrails](#guardrails) |
| Release                              | [Git Workflow → Releases](#releases), [Security](#security) |

---

## Architecture

Monorepo with two packages for Nanit baby camera Home Assistant integration:

```
custom_components/nanit/   ← HA integration (Python, async)
packages/aionanit/         ← Nanit API client library (published to PyPI)
tests/unit/                ← Integration tests (80% coverage threshold)
dev/                       ← Docker-based dev HA instance
tools/                     ← CLI utilities (login, events, probe)
docs/                      ← Security checklist, connection reliability, testing
```

**Data flow:**
- **Push sensors**: Camera → WebSocket → `NanitCamera.subscribe()` → `NanitPushCoordinator` → entities
- **Cloud events**: `NanitCloudCoordinator` polls `GET /babies/{uid}/messages` every 30s
- **Camera stream**: `camera.stream_source()` returns RTMPS URL with fresh access token
- **Commands**: Entity → `NanitCamera` → WebSocket → camera

**Multi-camera**: One config entry per Nanit account (unique_id = email). `NanitHub` auto-discovers all babies/cameras. Entity unique IDs: `{camera_uid}_{key}`.

### Key files

| File | Purpose |
|------|---------|
| `__init__.py` | Entry setup/unload/migrate, `NanitData` dataclass |
| `hub.py` | `NanitHub` lifecycle, `CameraData` per-camera grouping |
| `config_flow.py` | Credentials + MFA, reauth, per-camera IP options |
| `coordinator.py` | `NanitPushCoordinator` (WebSocket push) + `NanitCloudCoordinator` (polling) |
| `entity.py` | `NanitEntity` base class with availability logic |
| `camera.py`, `sensor.py`, `binary_sensor.py`, `switch.py`, `number.py` | Entity platforms |
| `manifest.json` | Version, requirements, HA metadata |
| `aionanit/camera.py` | `NanitCamera` state machine, subscribe, commands |
| `aionanit/auth.py` | `TokenManager` (auto-refresh, token change callback) |
| `aionanit/ws/transport.py` | `WsTransport` (WebSocket connection, reconnect, keepalive) |

---

## Code Standards

- **Python**: 3.12+ target. Fully async — no blocking I/O in the event loop.
- **Linter**: Ruff (rules: B, BLE, C4, D, E, F, I, ICN, N, PGH, PIE, RUF, SIM, T20, UP, W). Line length: 100.
- **Type checking**: mypy strict mode. All functions must have type hints.
- **Formatting**: Ruff formatter (enforced via pre-commit).
- **Strings**: User-facing text in `strings.json` / `translations/en.json` — no hardcoded English.
- **Imports**: isort via Ruff. Known first-party: `aionanit`, `custom_components.nanit`.
- **Naming**: Follow existing patterns. Never change entity unique IDs or class names without a migration plan.
- **Tests**: New features must include tests. Coverage threshold: 80% (enforced in CI).

### Commands

```bash
just setup            # Install deps, tooling, pre-commit hooks
just check            # Run ALL checks (lint + format + typecheck + tests) — use before any PR
just fix              # Auto-fix lint issues and reformat
just test             # Integration tests with coverage (custom_components)
just test lib         # aionanit library tests
just test all         # Both test suites
just dev              # Start dev HA instance → http://localhost:8123
just dev restart      # Restart after code changes
just dev stop         # Stop dev HA instance
just release-retry    # Re-trigger release workflow after fixing CI (uses same tag)
just beta             # ⚠️  HUMAN ONLY — create beta pre-release → triggers PyPI publish
just beta v1.4.0-beta.1  # ⚠️  HUMAN ONLY — release a specific beta tag
just promote          # ⚠️  HUMAN ONLY — promote a beta to stable (interactive version picker)
just promote 1.4.0    # ⚠️  HUMAN ONLY — promote a specific version directly
```

---

## Git Workflow

### Repository settings (enforced on GitHub)

- **Branch protection** (`~ALL` ruleset): All branches require signed commits, a PR (no direct push), and passing CI status checks. No bypass actors.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wealthystudent/ha-nanit](https://github.com/wealthystudent/ha-nanit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
