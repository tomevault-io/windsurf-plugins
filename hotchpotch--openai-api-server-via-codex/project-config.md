---
trigger: always_on
description: This repository implements an OpenAI-compatible FastAPI server that forwards
---

# Repository Guidelines

## Project

This repository implements an OpenAI-compatible FastAPI server that forwards
`/v1/responses` and `/v1/chat/completions` requests through the local Codex
HTTP backend credentials.

Keep compatibility behavior aligned with the official `openai-python` client.
When changing request or response shapes, add or update tests that exercise the
client API rather than only raw HTTP payloads.

## Environment

- Use Python 3.10 or newer.
- Use `uv` for dependency management and command execution.
- The default foreground server binds to `127.0.0.1:18080`.
- Do not commit local virtualenv, cache, tox, pytest, or editor artifacts.
- The live integration tests use the machine's existing Codex authentication.
  Do not add real tokens or copied auth files to the repository.

## Development Commands

Run the full local validation suite before committing behavior changes:

```bash
uv run tox
```

GitHub Actions runs the same validation suite for pushes to `main` and for pull
requests through `.github/workflows/ci.yml`.

Run focused compatibility tests while iterating on request/response behavior:

```bash
uv run python -m pytest tests/test_openai_compat_server.py -q
uv run ruff check .
uv run ty check
```

Run real Codex backend integration tests only when live network/auth testing is
intended. These tests use the machine's Codex authentication and make real model
requests:

```bash
RUN_CODEX_LIVE_TESTS=1 uv run python -m pytest tests/test_live_integration.py -q -s
RUN_CODEX_LIVE_TESTS=1 uv run python -m pytest tests/test_live_codex_http_compatibility.py -q -s
```

Run the broad Codex HTTP OpenAI client compatibility matrix by itself when
investigating API surface regressions:

```bash
RUN_CODEX_LIVE_TESTS=1 uv run python -m pytest tests/test_live_codex_http_compatibility.py::test_live_codex_http_handles_openai_client_compatibility_matrix -q -s
```

Run the server locally:

```bash
uv run openai-api-server-via-codex
uv run openai-api-server-via-codex --port 18080
uv run openai-api-server-via-codex --verbose
uv run openai-api-server-via-codex --config ~/.config/openai-api-server-via-codex/config.toml
```

Generate a config template:

```bash
uv run openai-api-server-via-codex config-generate
uv run openai-api-server-via-codex config-generate --stdout
```

Validate package artifacts before a PyPI release:

```bash
uv run tox
rm -rf dist
uv build --no-sources
uv run twine check --strict dist/*
uv run --with "$(ls dist/*.whl)" --no-project openai-api-server-via-codex --help
```

Generate release note text from the draft or finalized release notes:

```bash
python scripts/release-notes.py vX.Y.Z
```

## Implementation Notes

- Keep the public API OpenAI-compatible for both sync-style and async-style
  `openai-python` usage.
- Support both non-streaming and `stream=true` flows for Responses and Chat
  Completions.
- Preserve `previous_response_id` behavior by updating the in-memory response
  store when a response completes, including streaming responses.
- Keep stored Chat Completions compatible with the `openai-python`
  `client.chat.completions.list/retrieve/update/delete` and
  `client.chat.completions.messages.list` APIs. Chat `metadata` is stored by
  this compatibility server and should not be forwarded to Codex backends unless
  the backend explicitly supports it.
- Keep Responses helper APIs compatible with the `openai-python`
  `client.responses.retrieve(..., stream=True)`,
  `client.responses.input_tokens.count`, `client.responses.delete`, and
  `client.responses.cancel` call shapes.
- The only backend is `codex-http`. The previous native Codex app-server
  backend was removed because it was unstable; do not keep compatibility paths
  for it unless it is deliberately reimplemented later.
- Normalize Codex HTTP backend requests at the backend boundary: force the
  downstream Codex call to `stream=true` and `store=false`, default text
  verbosity to `low`, default `tool_choice`/`parallel_tool_calls`, include
  `reasoning.encrypted_content`, and add Codex-compatible stream headers.
  Public `store=true` compatibility is handled by local in-memory stores, not
  by forwarding `store=true` to ChatGPT Codex.
- Config is loaded from `--config`, `OPENAI_VIA_CODEX_CONFIG`, or the XDG path
  `$XDG_CONFIG_HOME/openai-api-server-via-codex/config.toml`, falling back to
  `~/.config/openai-api-server-via-codex/config.toml`. Setting precedence is
  CLI flag, environment variable, config file, default.
- Daemon PID and log files default under the config directory's `run/`
  subdirectory. `start`, `stop`, and `status` should all resolve the same
  config-backed daemon paths. `stop` and `status` should accept `--verbose`.
  When host is omitted and the exact default PID file is absent, they may
  discover a single PID file for the selected port; if multiple PID files
  match, they must refuse to guess and ask for `--host` or `--pid-file`.
- In-memory compatibility stores are intentionally bounded. Keep
  `max_stored_items` defaulting to 1000 and apply it consistently to
  `ResponseStore` and `ChatCompletionStore`. Evict oldest entries first; `0`
  means no in-memory storage.
- Codex backend request concurrency is intentionally bounded. Keep

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hotchpotch/openai-api-server-via-codex](https://github.com/hotchpotch/openai-api-server-via-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
