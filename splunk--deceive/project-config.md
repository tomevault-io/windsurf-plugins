---
trigger: always_on
description: This document gives AI coding agents the project-specific context needed to make
---

# AGENTS.md - DECEIVE

This document gives AI coding agents the project-specific context needed to make
consistent, idiomatic changes to DECEIVE.

## Project Overview

DECEIVE, the DECeption with Evaluative Integrated Validation Engine, is a
proof-of-concept high-interaction SSH honeypot. It accepts SSH connections,
authenticates according to configurable honeypot account rules, sends attacker
input to a configured LLM backend, returns realistic Linux-like command output,
and writes JSON Lines telemetry for the full session.

The LLM-backed SSH interaction is DECEIVE's core simulation surface. The core
engineering goal is to make that live interaction observable, bounded, testable,
and believable without exposing a real shell or real filesystem.

Primary implementation files:

- `SSH/ssh_server.py` - AsyncSSH server, authentication, prompt assembly,
  LangChain message history, JSON logging, and runtime configuration.
- `SSH/config.ini.TEMPLATE` - tracked operator configuration template.
- `SSH/prompt.txt` - default user prompt describing the host being emulated.
- `tests/` - unit and integration coverage for configuration, authentication,
  logging, session behavior, and real AsyncSSH connectivity with a fake LLM.
- `README.md` - user-facing setup, runtime, testing, and log format reference.
- `TODO.txt` - lightweight backlog and known priorities.

## Start-Of-Work Checklist

For any non-trivial change:

1. Read `README.md`, `pyproject.toml`, and `TODO.txt` before editing.
2. Check `git status --short`; preserve user changes already in the tree.
3. Inspect the relevant tests before changing behavior.
4. Update `TODO.txt` only when the change completes, changes, or adds a tracked
   backlog item.
5. Update `README.md` and `SSH/config.ini.TEMPLATE` when setup, config keys,
   runtime behavior, or log fields change.

## Tech Stack

- Python 3.11, pinned by `.python-version` and `requires-python` in
  `pyproject.toml`.
- `uv` for dependency management and command execution.
- `asyncssh` for the SSH server and integration test clients.
- LangChain provider integrations for OpenAI, Azure OpenAI, Ollama, AWS Bedrock,
  and Google Gemini.
- `pytest` and `pytest-asyncio` for automated tests.
- Standard library `argparse`, `configparser`, `logging`, `json`, `asyncio`, and
  path utilities. The current CLI is argparse-based; do not switch frameworks
  unless explicitly requested.

## Dependency Management

Use `uv`; do not add `requirements.txt` or install dependencies with bare `pip`.

Common commands:

```bash
uv sync
uv run pytest
uv run pytest tests/test_ssh_server_unit.py
uv run pytest tests/test_ssh_integration.py
uv run python SSH/ssh_server.py
```

When adding or removing dependencies, update `pyproject.toml` and `uv.lock`
together. This project currently has `package = false`, so treat it as a script
repository rather than an installed Python package.

## Code Style

- Prefer clear, direct Python over clever abstractions.
- Add type hints for new or significantly changed functions. Existing code is
  still being modernized, so avoid broad type-only churn.
- Use specific exceptions and actionable error messages at runtime boundaries.
- Avoid bare `except Exception` in new code unless it is at `main()` or another
  intentional process boundary.
- Prefer `pathlib.Path` for new path-heavy code, but match nearby code when a
  small change in `SSH/ssh_server.py` would otherwise create needless churn.
- Keep lines readable, around 100 characters where practical.
- Use comments sparingly for non-obvious async, logging, or security behavior.
- Do not add linting or formatting tool mandates unless the project config is
  updated to support them.

## Runtime Architecture

### SSH Server

`start_server()` creates an AsyncSSH listener from the active config. Preserve
these behavior contracts:

- `listen_host` may constrain binding; tests use `127.0.0.1`.
- `port = 0` must work in tests to request a random local port.
- Host private keys are resolved relative to the loaded config file first, then
  relative to `SSH/`.
- The server version string intentionally imitates OpenSSH.
- The process handler must never grant access to a real local shell.

`MySSHServer` owns SSH connection/auth callbacks. `handle_client()` owns
interactive and non-interactive command handling. There is known cleanup work in
`TODO.txt` around lifecycle ownership; avoid deepening the split between server
instances and process handling.

### Authentication Semantics

The honeypot intentionally supports deceptive login modes:

- `username =` accepts login without a password.
- `username = secret` requires the exact password.
- `username = *` accepts any password, including empty passwords.
- Unknown usernames currently authenticate like wildcard accounts.

Do not "fix" the unknown-user behavior unless implementing an explicit auth
policy option. Tests should cover all four modes.

### LLM Simulation

`build_message_history()` composes:

1. The configured system prompt from `[llm].system_prompt`.
2. The user prompt from `--prompt`, `--prompt-file`, or `SSH/prompt.txt`.
3. Per-session message history trimmed to `trimmer_max_tokens`.

Preserve per-session isolation through `llm_sessions` and the session id passed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [splunk/DECEIVE](https://github.com/splunk/DECEIVE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
