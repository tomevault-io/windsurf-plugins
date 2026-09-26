---
trigger: always_on
description: `app/` contains the Python service. `main.py` assembles FastAPI and FastMCP; `routes/` handles OpenAI, Twilio, debug, and deployment callbacks through explicit `CallService` methods (routes never touch `service.db` directly); `call_state.py` (the `CallService` facade and call state machine), `call_activity.py` (liveness/heartbeat tracking), `owner_transfer.py` (the owner-transfer saga), `twilio_bridge.py`, and `openai_live.py` coordinate live calls; and the `db/` package (a `Database` facade com
---

# Repository Guidelines

## Project Structure & Module Organization

`app/` contains the Python service. `main.py` assembles FastAPI and FastMCP; `routes/` handles OpenAI, Twilio, debug, and deployment callbacks through explicit `CallService` methods (routes never touch `service.db` directly); `call_state.py` (the `CallService` facade and call state machine), `call_activity.py` (liveness/heartbeat tracking), `owner_transfer.py` (the owner-transfer saga), `twilio_bridge.py`, and `openai_live.py` coordinate live calls; and the `db/` package (a `Database` facade composed from per-concern modules: engine, plans, deployment, calls, transfers, termination, telemetry, webhooks, transcripts, questions), `models.py`, `policy.py`, and `security.py` own persistence, schemas, call policy, and authentication. Tests mirror these concerns in `tests/test_*.py`, with shared fixtures in `tests/conftest.py`. Operational files live at the repository root (`Dockerfile`, `fly.toml`, `render.yaml`), while `scripts/run_sip_canary.py` performs live end-to-end validation and `scripts/live_smoke.sh` boots the app with dummy credentials for a local MCP smoke test.

## Build, Test, and Development Commands

- `uv sync --all-groups --frozen`: install dependencies from `uv.lock`.
- `uv run uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload`: run the local service with reload.
- `uv run ruff format --check app tests scripts`: verify formatting.
- `uv run ruff check app tests scripts`: run configured lint and import checks.
- `uv run mypy app`: run strict mypy type checking on the application package.
- `uv run pre-commit install`: install local git hooks (ruff, large-file check, gitleaks, mypy).
- `uv run pre-commit run --all-files`: run the same hooks on the full tree.
- `uv run pytest -q`: run the automated test suite.
- `uv run pytest -q tests/test_security.py`: run one focused test module.
- `uv run python scripts/run_sip_canary.py --mode full`: validate a deployed SIP flow; this places a real call and requires configured credentials.

## Coding Style & Naming Conventions

Use four-space indentation, Python type hints, and `from __future__ import annotations`. Ruff targets Python 3.12 with a 100-character line length and enforces pycodestyle, Pyflakes, import sorting, pyupgrade, bugbear, and async rules. Use `snake_case` for modules, functions, variables, and tests; `PascalCase` for classes and Pydantic models; and descriptive async names for network or database operations.

## Voice latency is a completion requirement

Latency is a first-order product requirement for this voice agent, alongside factual correctness and call safety. Treat an awkward multi-second pause as an unresolved defect, even when the call eventually ends and automated tests pass.

- Keep grounded acknowledgements and normal speech off unnecessary tool, database, and extra model round trips. Never invent an answer or acknowledge unheard speech to make latency look better.
- Before changing a conversational path, establish a latency target and measure from the callee's actual speech end to the first audio they hear. Separate turn detection/transport delay, model time, tool handling, and playback; first text and generation completion are not audible response time.
- Verify natural timing as well as factual answers, genuine interruptions, completed playback, post-goodbye follow-ups, and termination. Include clean and relevant noisy conditions; disclose incomplete scenarios and distinguish local transport simulation from real-phone evidence.
- Do not call a voice change done while a known latency problem remains. Keep working on it, or clearly state the unresolved limitation. Do not shorten the post-goodbye reply window or cut off speech to conceal an earlier response delay.

## Testing Guidelines

Tests use pytest, `pytest-asyncio` in auto mode, `respx`, and temporary SQLite databases. Name files `test_<area>.py` and tests `test_<behavior>`. Add regression coverage for state transitions, signed webhook handling, payload shapes, recovery, and teardown. CI enforces an 85% coverage floor on `app` via `pytest --cov=app`; new behavior should still exercise both success and failure paths.

## Commit & Pull Request Guidelines

Recent history uses short, imperative, sentence-case subjects such as `Add call-safe deployment lease`. Keep commits focused. Pull requests should explain the behavioral change, risks to live-call teardown or billing, configuration changes, and validation performed. Link relevant issues; include logs or payload examples for backend changes and canary evidence when telephony behavior changes. Ensure Ruff and pytest pass before review.

## Security & Operations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XiyaoWang0519/agent-call](https://github.com/XiyaoWang0519/agent-call) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
