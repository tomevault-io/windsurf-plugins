---
trigger: always_on
description: Validates a set of versions against known compatibility matrices and returns `{compatible, conflicts[], notes[]}`. Conflicts carry `kind`, `requested`, `expected`, `suggestion`, and `reference`.
---

# AGENTS.md

This file is the project guide for coding agents (Claude Code, Cursor, Codex, Grok Build, and others). [CLAUDE.md](CLAUDE.md) only points here. Install and day-to-day setup live in [README.md](README.md) and [docs/configuration.md](docs/configuration.md).

## Non-negotiables

Rules that are not open for discussion. Violating these is an error, not a judgment call.

- **No XML parser dependency.** All XML parsing is regex-based — avoids a heavyweight dependency for the small subset of XML used in Maven metadata and POM files.

## Project

MCP server for Maven dependency intelligence. Provides tools to query artifact versions from Maven repositories (Maven Central, Google Maven, Gradle Plugin Portal).

**Implementation:** `plugin/server/server.py` — a single-file Python 3 server (stdlib only, zero pip dependencies). It speaks MCP over stdio (JSON-RPC 2.0 on stdin/stdout) or, when `MAVEN_MCP_TRANSPORT=http`, over a stateless Streamable HTTP endpoint (`POST /mcp`, stdlib `http.server.ThreadingHTTPServer`, JSON responses, no SSE/sessions). The stdio mode is registered via `plugin/.claude-plugin/plugin.json` as `command: python3`. Runs as an MCP server in local and cloud agent environments without Node.js or npm, and standalone with any MCP-compatible client.

**Stack:** Python 3.9+ standard library only (`urllib`, `json`, `re`, `typing`). No build step, no install step.

## Commands

All commands run from the repository root.

```bash
python3 -m unittest discover -s tests       # Run all tests
python3 -m unittest discover -s tests -v    # Verbose
python3 -m compileall plugin/server         # Zero-dep syntax gate
```

`tests/test_wheel.py` builds a wheel, so that interpreter needs hatchling
(`pip install 'hatchling>=1.26.3'`). CI installs it in `python-tests` and
`coverage`. The server runtime stays dependency-free.

Run a single test module:

```bash
python3 -m unittest discover -s tests -p test_handlers.py
```

**Lint / type-check / coverage (#408).** Dev-only tools, not runtime dependencies —
install ephemerally (`pip install ruff==0.15.22 mypy==1.20.2 coverage==7.15.2`, or a
venv). Config lives in `pyproject.toml` (`[tool.ruff]`,
`[tool.mypy]`, `[tool.coverage.*]`); see its comments for what is/isn't enabled and
why (pragmatic ruleset, per-file-ignores for pre-existing findings, mypy targets the
3.9 floor independent of the interpreter running it).

```bash
ruff check plugin/server tests
mypy --config-file pyproject.toml
coverage run --rcfile=pyproject.toml -m unittest discover -s tests
coverage report --rcfile=pyproject.toml   # fail_under=75, ~90% measured
```

## Work and releases

Ordinary changes keep the version already on `main`. A release is a separate cut, and only when the user asks for one. `1.0.0` is published: git tag `v1.0.0` is `02fa8f5` on `main`, and PyPI project `maven-mcp` serves that version. Do not rebuild, retag, or re-upload it.

### Day-to-day

- Issues belong in `kirich1409/maven-mcp`.
- Branch from `main` as `feature/…`, `fix/…`, or `chore/…` (kebab-case English). `main` rejects a direct push: rulesets Main Protect (no deletion, no non-fast-forward, linear history) and Pull Request Only.
- Open a pull request. Required checks are `python-tests (3.9)`, `python-tests (3.13)`, `ruff`, and `mypy`. Coverage runs in CI and is not required. Review count is 0; review threads must be resolved; Copilot reviews on push. Squash-merge and delete the branch. Merge and rebase are allowed; squash is the history this repo keeps.
- When the user asks for a PR, push the branch, open it, wait until the required checks are green, and squash-merge. Do not leave the PR open.
- Leave `SERVER_VERSION`, `USER_AGENT`, the three plugin manifests, `.claude-plugin/marketplace.json`, and `[project].version` on the current release until the release cut below. `python3 scripts/check-versions.py` must exit 0.
- A new runtime dependency needs an explicit yes. hatchling stays a build-system dependency. The wheel's `force-include` keeps `compat-matrices.json` beside the installed `server.py`.
- Do not commit `.grok-report/`, build artifacts, or credentials.

### Cutting a release

1. One PR sets a new `X.Y.Z` in every place `scripts/check-versions.py` compares, then lands on `main` with the required checks green. Patch for a fix, minor for compatible behavior. `python3 scripts/check-versions.py` exits 0 on that commit.
2. `tests/test_wheel.py` still passes: the wheel contains `server.py` and `compat-matrices.json` at install root, `License-Expression: MIT`, the README text, `Requires-Python: >=3.9`, and no `Requires-Dist`. PyPI JSON then reports `license_expression` `MIT` and leaves legacy `license` null.
3. Tag that merge commit `vX.Y.Z` and create the GitHub Release on the same commit. The monorepo-era name `maven-mcp--vX.Y.Z` is retired. Do not move a tag that already points at a published commit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kirich1409/maven-mcp](https://github.com/kirich1409/maven-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
