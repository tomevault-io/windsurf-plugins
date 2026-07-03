---
trigger: always_on
description: See @README.md for project overview and @pyproject.toml for dependencies and build config.
---

# identark — Claude Code Instructions

See @README.md for project overview and @pyproject.toml for dependencies and build config.

## What This Project Is

identark implements the **AgentGateway Protocol (AGP)** — a credential isolation layer for
production AI agents. Agents never hold raw credentials. Every tool call routes through a gateway
that fetches credentials, executes, logs, and returns results. The agent only ever sees an agent ID.

This is both an open-source Python SDK and the foundation of identark-cloud (hosted SaaS).

---

## Build & Test Commands

```bash
# Install in dev mode
pip install -e ".[dev]"

# Lint (MUST pass before any commit)
ruff check identark/ tests/

# Type check (MUST pass before any commit)
mypy identark/

# Run unit tests
pytest tests/unit/ -v --cov=identark --cov-report=term-missing

# Run all tests
pytest -v

# Build distribution
python -m build

# Format code
ruff format identark/ tests/
```

IMPORTANT: Always run `ruff check` and `mypy` after making changes. Never leave type errors unresolved.

---

## Project Structure

```
identark/
├── __init__.py              # Public API exports
├── gateway.py               # Abstract base: IdentArkGateway
├── models.py                # Pydantic models: AgentConfig, GatewayRequest, GatewayResponse
├── exceptions.py            # All custom exceptions
├── py.typed                 # PEP 561 marker
├── gateways/
│   ├── __init__.py
│   ├── direct.py            # DirectGateway — local, no network, for dev/testing
│   └── control_plane.py     # ControlPlaneGateway — calls identark-cloud API
├── integrations/
│   ├── __init__.py
│   ├── langchain.py         # LangChain tool wrapper
│   ├── n8n.py               # n8n node bridge (Python side)
│   └── crewai.py            # CrewAI agent wrapper
└── testing/
    ├── __init__.py
    └── mock_gateway.py      # MockGateway for unit tests — no real credentials needed

tests/
├── conftest.py              # Shared fixtures: mock_gateway, sample_agent_config
├── unit/
│   ├── test_models.py
│   ├── test_exceptions.py
│   ├── test_gateways.py
│   └── test_integrations.py
└── integration/             # Requires live identark-cloud (skip in CI by default)
```

---

## Architecture Rules — NEVER VIOLATE THESE

- **Agents NEVER hold raw credentials.** No API keys, tokens, or secrets in agent context, tool
  definitions, or environment variables accessible to the agent.
- **All tool calls go through a gateway.** DirectGateway for local dev, ControlPlaneGateway for cloud.
- **Every gateway execution produces an audit log entry.** Log: agent_id, tool, timestamp,
  success/failure, latency. Never log credential values, not even partially.
- **Credentials are fetched per-execution, never cached in memory across requests.**
- **Multi-tenant isolation is enforced at the gateway layer.** Agent A cannot access credentials
  registered under Agent B, even within the same organisation.

---

## Code Style

- Python 3.10+ only. Use `X | Y` union syntax, not `Optional[X]` or `Union[X, Y]`.
- All public functions and classes MUST have type annotations. mypy strict mode is enforced.
- Use Pydantic v2 models for all data structures (already a dependency via httpx).
- Async-first: all gateway operations are `async`. Provide sync wrappers only where explicitly needed.
- Exception hierarchy: all exceptions inherit from `IdentArkError` in `exceptions.py`.
- Use `httpx.AsyncClient` for all HTTP calls. Never use `requests`.
- Line length: 100 characters (configured in pyproject.toml).

---

## Security Requirements

- NEVER log, print, or include credential values in exceptions, error messages, or stack traces.
- NEVER store credentials in Pydantic model fields that could be serialised to JSON without
  explicit `exclude=True`.
- NEVER write tests that use real API keys. Use `MockGateway` or `pytest-mock`.
- If a function receives a credential value, it must be typed as `SecretStr`, not `str`.
- All credential storage in the cloud layer uses HashiCorp Vault or AWS Secrets Manager.
  NEVER store credentials in PostgreSQL plaintext columns.

---

## Testing Conventions

- Every new gateway method needs a corresponding unit test in `tests/unit/test_gateways.py`.
- Use `MockGateway` from `identark.testing` for all tests that would otherwise need real credentials.
- Async tests use `pytest-asyncio` with `asyncio_mode = "auto"` (already configured).
- Test file naming: `test_<module>.py` mirroring the source path.
- Aim for >90% coverage on `identark/` core. Integrations can be lower.
- Mark integration tests with `@pytest.mark.integration` so CI can skip them:
  `pytest tests/unit/ -v` for CI, `pytest -v -m "not integration"` locally.

---

## Git & PR Conventions

- Branch naming: `feat/<name>`, `fix/<name>`, `chore/<name>`, `docs/<name>`
- Commit messages: conventional commits — `feat:`, `fix:`, `docs:`, `chore:`, `test:`
- NEVER commit directly to `main`. All changes via PR.
- Run `ruff check` and `pytest tests/unit/` before every commit.
- Tag releases as `vX.Y.Z` — this triggers the PyPI publish workflow in CI.

---

## identark-cloud (Cloud Control Plane)

The hosted SaaS layer lives in `cloud/` (to be scaffolded). Stack:

- **API:** FastAPI, async, Python 3.11+

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [identArk/identark](https://github.com/identArk/identark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
