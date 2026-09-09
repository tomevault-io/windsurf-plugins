---
trigger: always_on
description: Always run `uv run ruff format --check .` and `uv run python -m pytest tests/ -q` after making changes.
---

# Development

Always run `uv run ruff format --check .` and `uv run python -m pytest tests/ -q` after making changes.

# Publishing

Bump `version` in `pyproject.toml` before creating a release. The publish workflow triggers on GitHub releases, not tags alone.

---
> Source: [bruin-data/python-sdk](https://github.com/bruin-data/python-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
