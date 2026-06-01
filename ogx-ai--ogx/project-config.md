---
trigger: always_on
description: This file provides guidance for AI coding agents working on this codebase.
---

# Agent Guidelines for OGX

This file provides guidance for AI coding agents working on this codebase.
It complements [CONTRIBUTING.md](CONTRIBUTING.md) with agent-specific instructions.
Human contributors should follow the conventions in CONTRIBUTING.md.

## Project Overview

OGX is an API server implementing the OpenAI Responses API, Chat Completions,
Embeddings, and supporting APIs (files, vector stores, batches, eval, and responses guardrails). It supports
multiple inference backends (OpenAI, Azure, Bedrock, vLLM, Ollama, WatsonX, etc.) through
a provider architecture.

## Repository Layout

```text
src/ogx/              # Server implementation
  core/                       # Request routing, server, storage
  providers/
    inline/                   # Built-in providers (responses, eval, vector_io, etc.)
    remote/                   # Remote provider adapters (OpenAI, Azure, vLLM, etc.)
    registry/                 # Provider registration specs
    utils/                    # Shared provider utilities (OpenAI mixin, MCP, etc.)
  distributions/              # Distribution configs (starter, ci-tests, etc.)
src/ogx_api/          # API definitions, Pydantic models, FastAPI routes
tests/
  unit/                       # Unit tests
  integration/                # Integration tests with recording/replay system
```

## Python & Tooling

- **Python 3.12** is required. Pre-commit hooks only work with 3.12.
- Use `uv` for all dependency management and script execution.
- Run scripts with `uv run`, never bare `python3` or `python`.
- Use standard library modules when possible.
- All function signatures must have type hints. Prefer Pydantic models for validation.
- Code must pass `mypy`. Check the exclude list in `pyproject.toml` for known exceptions.
- Use `def _function_name` for private functions.
- Prefer explicit top-level imports over inline imports.
- Do not use exceptions as control flow.

## Code Style

- Comments must add value. Do not write filler comments that describe the next line.
- Add comments only to clarify surprising behavior that is not obvious from the code.
- Good variable naming and clear code organization matters more than comments.
- Do NOT remove existing comments unless they are factually wrong.
- Error messages must be prefixed with "Failed to ...".
- Use structured logging via `from ogx.log import get_logger`. Always use
  key-value style: `logger.info("Processing request", model=model_id, provider=provider)`
  instead of f-strings or %-style formatting. The pre-commit hook
  `Block f-string logging` enforces this.
- The pre-commit hook `Ensure 'ogx.log' usage for logging` enforces that all
  logging uses the project's logger, not the standard library directly.

## Git Conventions

- Always use `--signoff` (`-s`) when creating commits.
- Do not amend commits and force push during PR review. Use new commits instead.
- Use `git merge main` to update branches, not `git rebase`.
- Commit messages must use [conventional commits](https://www.conventionalcommits.org/)
  format and full sentences, not bullet points.
- Merge `upstream/main` before pushing a branch to avoid CI failures from stale code.

## Testing

- Unit tests: `./scripts/unit-tests.sh` or `uv run pytest tests/unit/ -x --tb=short`
- Run pre-commit checks: `uv run pre-commit run --all-files`

### Integration Tests

Integration tests use a recording/replay system. Recordings are JSON files in
`tests/integration/*/recordings/` keyed by SHA256 hashes of HTTP request bodies.
When modifying code that changes request bodies sent to providers, recordings may
need to be re-recorded.

Run integration tests via the CI script (same command used in GitHub Actions):

```bash
# Replay mode (default) — uses pre-recorded responses, no API keys needed
uv run --no-sync ./scripts/integration-tests.sh \
  --stack-config server:ci-tests --setup gpt \
  --file tests/integration/responses/test_compact_responses.py

# Re-record missing recordings (requires API keys, e.g. OPENAI_API_KEY)
uv run --no-sync ./scripts/integration-tests.sh \
  --stack-config server:ci-tests --setup gpt \
  --inference-mode record-if-missing \
  --file tests/integration/responses/test_compact_responses.py

# Run a full suite
uv run --no-sync ./scripts/integration-tests.sh \
  --stack-config server:ci-tests --setup gpt --suite responses
```

Key flags: `--stack-config` (required), `--setup` (`gpt`, `ollama`, `vllm`),
`--inference-mode` (`replay`, `record`, `record-if-missing`), `--file` (single file),
`--pattern` (pytest `-k` filter), `--suite` (`base`, `responses`, `vision`).

If a test fails in replay mode with "Recording not found", re-run with
`--inference-mode record-if-missing` and commit the new recording files.
Recording locally requires an API key for the provider under test (e.g.
`OPENAI_API_KEY` for `--setup gpt`). If you do not have a key, repository
maintainers can trigger the record workflow on GitHub via
`.github/workflows/record-integration-tests.yml`.

## Provider Architecture

Each provider implements a protocol (e.g., `Inference`, `Responses`, `VectorIO`) and is
registered in `src/ogx/providers/registry/`. Provider specs include:

- `provider_type`: e.g., `remote::openai`, `inline::builtin`
- `module`: Python module path

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ogx-ai/ogx](https://github.com/ogx-ai/ogx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
