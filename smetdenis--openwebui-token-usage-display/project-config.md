---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A **single-file Open WebUI filter plugin**. The deliverable is `usage_display.py` (~2000 lines),
which a user pastes into Open WebUI **Admin → Functions**. There is **no build step and no
install** — everything else in the repo (tests, docs, CI) is scaffolding around that one file. It
is published on the community store (<https://openwebui.com/posts/token_usage_display_a94ea72f>)
and listed in the official Open WebUI docs Community Plugins catalog
(<https://docs.openwebui.com/features/extensibility/community/>).

## Critical rules

- **Keep it one copy-pasteable file with zero runtime dependencies.** Never split the plugin into
  modules. Its only optional imports (`tiktoken`, `aiohttp`) are **soft-imported** so the plugin
  loads without them.
- **Never add a `requirements:` line to the docstring frontmatter.** It triggers a per-plugin
  `pip install` inside OWUI that fails in uvx/LXC deployments and blocks the plugin from loading
  (real user bug reports) — soft imports exist precisely to avoid it.
- **Do not reflow the docstring frontmatter.** OWUI parses strictly one `key: value` per line
  (`utils/plugin.py:extract_frontmatter`); wrapping a long line truncates the metadata, and the
  first line must stay bare `"""`. This is why the closing `"""` carries
  `# noqa: D205, D212, D415, E501` (ruff reads a multi-line string's noqa from its last line).
- **Verify every claim about OWUI behavior against real OWUI source, not memory or docs.** The
  data model shifts between versions. A read-only local clone path and freshness-check commands
  live in `CLAUDE.local.md`; the verified map is in the imported `docs/owui-map.md`. If the clone
  is stale relative to the latest OWUI release, say so — the user updates it himself.

## Deep-dive references (imported, always loaded)

- Plugin internals — pipeline, valves, resolution chains, token/cache semantics, debug payload,
  edge-case catalog with covering tests: @docs/plugin-internals.md
- Open WebUI source map — filter machinery, request lifecycle, usage normalization, persistence,
  events, frontend constraints (verified against v0.11.1): @docs/owui-map.md

## Commands

Dev tooling is managed by **uv**: the `dev` dependency group in `pyproject.toml` plus the committed
`uv.lock` (exact versions). The `Makefile` runs every tool as `uv run --locked …`, which fails if the
lock is stale, so local runs and CI use identical ruff/mypy/pytest versions. There is no
`requirements-dev.txt` any more.

```bash
make install-dev                           # one-time setup: uv sync --locked (creates .venv)
make pre-commit                            # the full gate: lint + typecheck + test-cov
make lock        # re-lock after editing [dependency-groups]
make upgrade     # bump every locked tool to its latest allowed version (then run the gate)
make lint        # ruff check + ruff format --check
make format      # ruff format + ruff check --fix (auto-fix)
make typecheck   # mypy --strict
make test        # pytest
make test-cov    # pytest with coverage (term-missing + HTML in htmlcov/)
```

`make pre-commit` must be green before any change is done. It enforces **mypy strict** and a
**coverage floor of 95%** (`pyproject.toml` `fail_under = 95`). CI (`.github/workflows/ci.yml`,
single `gate` job) runs the same gate across Python 3.11–3.14 on pushes to `main` and all PRs,
installing tools with `astral-sh/setup-uv` (pinned by commit SHA — it publishes no floating major
tag) and `uv sync --locked`.

Run a single test: `uv run --locked pytest tests/test_usage_display.py::test_render_input`
(or `-k <substring>`). The suite has no network and no Open WebUI runtime — it is fast.

## Releasing a change

- The version has a **single source of truth**: the `version:` field in the `usage_display.py`
  module docstring. When bumping it, mirror it in `pyproject.toml` `version` and add a matching
  top entry in `CHANGELOG.md`. (`required_open_webui_version: 0.9.0` in the same docstring is the
  compat floor; native provider cost needs 0.10.0+.)
- **Tag each release `vX.Y.Z`** matching the CHANGELOG entry (existing convention: `v2.0.0` …
  `v2.6.0`); there is no automated release workflow — tagging is manual.
- **Do not reuse a published version number.** `v2.6.0` was re-cut on 2026-09-15 (owner's choice): the
  original `v2.6.0` and `v2.6.1` releases of 2026-09-14 were deleted and the unreleased 2.7.0/2.8.0 work
  folded into one `v2.6.0`. Accepted cost: two different builds carried `2.6.0`, so users who pasted the
  first one see no version change. Going forward, cut a new version instead.
- The community-store post text is maintained in `docs/community-post.md`; publishing to the store
  post (URL above) is a manual copy-paste done by the user.

## Architecture (summary — details live in the imports above)

- `class Filter` implements the OWUI filter contract: async `inlet` (stashes wall-clock start and
  a `num_ctx` hint) and async `outlet` (the orchestrator: gates → extract tokens → timing →
  context → cost → build stats → emit a `status` event). Both resolver calls are individually

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SmetDenis/openwebui-token-usage-display](https://github.com/SmetDenis/openwebui-token-usage-display) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
