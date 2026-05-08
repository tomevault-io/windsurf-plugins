---
trigger: always_on
description: - Repo: https://github.com/renan/ClawLite (update with real URL)
---

# AGENTS.md — ClawLite Repo Guide

- Repo: https://github.com/renan/ClawLite (update with real URL)
- In chat replies, file references must be repo-root relative only (example: `clawlite/channels/telegram.py:80`); never absolute paths.
- GitHub issues/comments/PR bodies: use literal multiline strings or `-F - <<'EOF'` for real newlines; never embed `\n` as text.
- GitHub comment footgun: never use `gh issue/pr comment -b "..."` when body contains backticks or shell chars. Always use `-F - <<'EOF'` heredoc.
- When working on a GitHub Issue or PR, print the full URL at the end of the task.
- When answering questions, respond with high-confidence answers only: verify in code; do not guess.
It captures the repo's actual commands, validation workflow, and coding style.

## Priority
1. Follow safety and applicable law.
2. Follow the user's explicit request.
3. Follow repository-specific guidance in this file.
4. Prefer small, auditable changes over broad refactors.

Core behavior inherited from the previous local guide:
- Be objective, technical, and useful.
- Ask questions only when critical information is missing.
- Prefer execution plus verifiable results.
- Validate with tests or smoke checks when practical.
- Explain change impact clearly.
- Never expose secrets or use destructive commands casually.

## Rule Files Found
- Root guidance exists in this `AGENTS.md`.
- No `.cursor/rules/` directory was found.
- No `.cursorrules` file was found.
- No `.github/copilot-instructions.md` file was found.

## Repo Snapshot
- Language: Python.
- Packaging: `pyproject.toml` with `setuptools`.
- Python: `>=3.10`.
- Package: `clawlite`.
- CLI entrypoint: `clawlite = clawlite.cli:main`.
- Gateway: FastAPI + uvicorn + WebSocket support.
- Runtime autonomy already includes supervised background loops for `heartbeat`, `autonomy`, `channels_dispatcher`, `channels_recovery`, `subagent_maintenance`, and `self_evolution`.
- Tests: `pytest`.
- Logging: `loguru` with helpers in `clawlite/utils/logging.py`.
- CI covers Python 3.10 and 3.12.

## Important Paths
- `clawlite/cli/` - commands, ops helpers, onboarding.
- `clawlite/gateway/` - FastAPI server and WebSocket endpoints.
- `clawlite/core/` - engine, prompt builder, memory, skills, subagents.
- `clawlite/runtime/` - autonomy, supervisor, self-evolution, runtime-side telemetry.
- `clawlite/scheduler/` - cron and heartbeat services.
- `clawlite/providers/` - provider registry, discovery, failover, LiteLLM glue.
- `clawlite/channels/` - channel adapters.
- `clawlite/tools/` - tool registry and tool implementations.
- `tests/` - pytest suite mirroring runtime modules.
- `scripts/` - smoke and release-preflight helpers.
- `docs/` - operational and API docs; update when behavior changes.

## Current Runtime Notes
- `clawlite/gateway/server.py` is the control-plane center for lifecycle, diagnostics, runtime loop startup/shutdown, and supervisor recovery wiring.
- `docs/API.md` must be updated whenever diagnostics payloads change. Current additive runtime payloads include `channels_dispatcher`, `channels_recovery`, `subagents`, and `self_evolution` runner telemetry.
- `tests/gateway/test_server.py` is the main regression file for gateway diagnostics, startup replay, and supervisor recovery behavior.
- `tests/channels/test_manager.py` covers dispatcher/recovery loop diagnostics and restart behavior for the `ChannelManager`.
- The autonomy loop already includes provider-aware suppression/backoff and a repeated-idle no-progress guard; preserve those semantics when extending autonomy.
- Config parsing must preserve explicit zero values (`0`, `0.0`) for cooldown/interval settings; avoid `raw or default` when `0` is a valid input.
- When testing startup replay/autonomy notices, disable the supervisor in that test if the expected first `channels.send(...)` call must come from replay instead of a recovery notice.
- For OpenClaw parity work, adapt behavior to ClawLite architecture; do not blindly copy files or structure from another repo.

## Setup
Recommended local setup:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -e .
```

If the repo later gains dev extras, prefer:

```bash
pip install -e ".[dev]"
```

## Build, Lint, and Test Commands
There is no dedicated build script in the repo; the normal loop is editable install + lint/test/smoke.

- Install editable: `pip install -e .`
- CLI help: `python -m clawlite.cli --help`
- Start gateway: `clawlite gateway --host 127.0.0.1 --port 8787`
- Alternate start: `clawlite start --host 127.0.0.1 --port 8787`
- Validate config: `clawlite validate config`
- Release preflight: `clawlite validate preflight --gateway-url http://127.0.0.1:8787`
- Scripted preflight: `bash scripts/release_preflight.sh --config ~/.clawlite/config.json --gateway-url http://127.0.0.1:8787`
- Smoke test: `bash scripts/smoke_test.sh`

Lint commands used by the repo today:
- CI baseline: `ruff check clawlite/ --select E9,F --ignore F401,F811`
- Runtime self-check baseline: `python -m ruff check --select=E,F,W .`

Testing commands used by the repo today:
- Full suite: `python -m pytest tests -q --tb=short`
- Coverage: `python -m pytest tests/ -q --tb=short --cov=clawlite --cov-report=term-missing --cov-report=xml:coverage.xml`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eobarretooo/ClawLite](https://github.com/eobarretooo/ClawLite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
