---
trigger: always_on
description: - Always use conventional commit standard for creating commit messages.
---

# Repository Instructions

- Always use conventional commit standard for creating commit messages.
- Prefix shell commands with `rtk`; use `rtk proxy` when unfiltered output is needed.
- Prefer `rg` and `rg --files` for code search. Use `uv` for project commands.
- Keep docs truthful to implemented behavior. Check command help and source before documenting CLI options or Python APIs.
- Preserve unrelated working-tree changes. Stage only the files belonging to the requested work.

## Repo Map

- `canfar/models/` contains Pydantic models for persisted config, Authentication Records, server metadata, registry data, and Session requests/responses.
- `canfar/models/config.py` defines `Configuration`; `canfar/config/editor.py` owns validated dotted edits and atomic persistence through `config.editor`.
- `canfar/authentication.py` owns public credential operations and Python `login`/`alogin`; `canfar/auth/` implements X.509 and OIDC flows. Shared OIDC credential initialization lives in `canfar/auth/oidc.py`.
- `canfar/server.py` owns public Server Selection and discovery operations; `canfar/_server_discovery.py` implements discovery and metadata enrichment. Selection policy does not live on `Configuration`.
- `canfar/client.py` composes native sync/async `httpx2` clients, credentials, auth hooks, and timeouts. Runtime credential precedence applies to hooks as well as headers and TLS.
- `canfar/sessions.py`, `canfar/images.py`, `canfar/context.py`, and `canfar/overview.py` expose Science Platform operations.
- `canfar/storage.py` resolves configured Storage Identifiers into fsspec filesystems; `canfar/cli/data.py` integrates the delegated storage CLI.
- `canfar/cli/` contains Typer adapters. `machine.py` declares shared options (leaf output, model-derived ranges); `output.py` renders structured output, and `output.fail()` reports a command failure in the selected mode and exits. `main.py` registers root commands in `_COMMANDS` as `module:attribute` targets that load only when used; keep feature-only dependencies (Authlib, segno, fsspec) and the HTTP stack (`httpx2`, `canfar.authentication`, `canfar.server`, `canfar.client`) out of import-time paths that every command takes. `tests/test_cli_startup.py` guards startup imports.
- `canfar/hooks/` and `canfar/utils/` contain HTTP/Typer hooks, discovery utilities, logging, and request builders.
- `tests/` mirrors source modules; `tests/conftest.py` isolates the test home and CANFAR environment.
- `docs/` is the MkDocs site; `mkdocs.yml` defines navigation. `skills/canfar/SKILL.md` is the user-facing operations skill, separate from repository engineering guidance.

## Validation

Use these project commands with the `rtk` prefix. Set `UV_CACHE_DIR=/tmp/canfar-uv-cache` if the default cache is unavailable.

- Fast lint: `rtk proxy uv run --no-sync ruff check . --no-cache`
- Type check: `rtk proxy uv run --no-sync ty check canfar`
- Deterministic non-slow tests: `rtk proxy uv run --no-sync pytest tests -m "not slow" --no-cov -q -o cache_dir=/tmp/canfar-pytest-cache`
- Docs build: `rtk proxy uv run --group docs mkdocs build`
- Full test suite: `rtk proxy uv run --no-sync pytest`
- Skill and docs truth: `rtk proxy uv run --no-sync pytest tests/test_skills.py --no-cov -q -n0`

Run focused tests for changed behavior before broader validation. Use the deterministic suite for work without live credentials. The full suite contacts CANFAR and creates/deletes test Sessions; run it only with a valid account, usable Authentication Record, and certificate.

Pytest creates an empty temporary home by default, so a certificate in your normal home alone does not configure integration tests. For an authenticated run, set `CANFAR_TEST_HOME` to a prepared temporary home containing a private copy of the test configuration and certificate. Keep the developer's normal configuration untouched and remove the temporary credential copies afterward.

Tests constructing `Configuration` must isolate `CONFIG_PATH`, even when using `model_validate`: settings sources can merge a populated configuration into explicit input. Report deterministic results separately from live service failures; a docs build or local test pass does not establish live workflow success.

## Agent skills

### Published skill

`skills/canfar/` is the one skill users install (`npx skills add opencadc/canfar`). A change to the CLI or Python surface updates `docs/` and the skill in the same change; `tests/test_skills.py` fails on commands, Python examples, API names, and links that drift. Put new platform knowledge in `docs/` and point the skill at it. See `docs/agents/skill.md`.

### Issue tracker

Issues, PRDs, and implementation decisions can be tracked in Jira on `herzberg.atlassian.net` (project `CADC`, label `CANFAR`) or GitHub Issues in `opencadc/canfar`. Follow the user's choice or the existing work item's tracker; ask which to use when neither resolves the destination. See `docs/agents/issue-tracker.md`.

### Triage labels

Map the five canonical skill roles to Jira statuses or GitHub issue labels according to the work item's tracker. Keep the `CANFAR` label on Jira work. See `docs/agents/triage-labels.md`.

### Domain docs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opencadc/canfar](https://github.com/opencadc/canfar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
