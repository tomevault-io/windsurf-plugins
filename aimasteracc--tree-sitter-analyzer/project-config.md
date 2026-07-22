---
trigger: always_on
description: > **Discovery path**: read this file → skim [`CLAUDE.md`](CLAUDE.md) for the locked design decisions → load only the [`docs/CODEMAPS/`](docs/CODEMAPS/) map matching the area you're touching. Do **not** load the full source tree blindly — the codemaps exist to keep agent context lean.
---

# Agent Instructions

> **Discovery path**: read this file → skim [`CLAUDE.md`](CLAUDE.md) for the locked design decisions → load only the [`docs/CODEMAPS/`](docs/CODEMAPS/) map matching the area you're touching. Do **not** load the full source tree blindly — the codemaps exist to keep agent context lean.

## Codemap Index

| Area | Codemap |
|---|---|
| High-level topology | [`docs/CODEMAPS/architecture.md`](docs/CODEMAPS/architecture.md) |
| MCP tools (count in codemap) | [`docs/CODEMAPS/mcp-tools.md`](docs/CODEMAPS/mcp-tools.md) |
| CLI flags / commands | [`docs/CODEMAPS/cli.md`](docs/CODEMAPS/cli.md) |
| Language plugins (count in codemap) | [`docs/CODEMAPS/languages.md`](docs/CODEMAPS/languages.md) |
| Output formatters | [`docs/CODEMAPS/formatters.md`](docs/CODEMAPS/formatters.md) |
| Security boundary | [`docs/CODEMAPS/security.md`](docs/CODEMAPS/security.md) |

## Test Runtime Contract

- The default full-suite command is `uv run pytest -q`.
- Do not run the full suite serially. Project pytest config enables xdist with `--numprocesses=auto --dist=loadfile`.
- The full suite must finish in under 5 minutes. The config enforces `--session-timeout=900` and `--timeout=30`. (Bumped from 300 in v1.13.1 — see `docs/POSTMORTEM_v1.13.md` § 9.)
- After edits, run `uv run python -m tree_sitter_analyzer --change-impact --format json` and follow its `verification_command`.
- If `test_required` is `false`, do not run tests just to look busy; run the reported non-test verification such as `git diff --check`.
- For targeted code feedback, prefer `verification_command`/`test_command`; `pytest_required` and `pytest_command` are retained for pytest-specific compatibility.
- For PRs that change Python source, run focused tests with `--cov=tree_sitter_analyzer --cov-report=json`, then run `uv run python scripts/check_patch_coverage.py --base origin/develop --coverage-json coverage.json` before pushing. The local patch gate must report no added executable misses; add effective tests instead of waiting for CI Codecov to block the PR.
- Benchmark-only runs are the exception: use `uv run pytest tests/benchmarks/ --benchmark-enable --benchmark-only -n 0 --session-timeout=0`.
- Do not remove or weaken these pytest defaults. They prevent repeated agent mistakes: serial full-suite runs, accidental benchmark execution, hidden hangs, and >5 minute feedback loops.
- If a test-runtime setting must change, update `tests/contracts/test_pytest_runtime_contract.py`, explain why the new setting is faster or safer, and prove `uv run pytest -q` still finishes under 5 minutes.

## CI Test Tier Contract

- CI must not run exhaustive all-language golden/regression tests on every OS/Python axis. Mark that class with `@pytest.mark.full_language`.
- `.github/workflows/reusable-test.yml` runs `full_language` tests only on the single Linux coverage axis; every no-coverage matrix job must exclude `not full_language`.
- PR feedback uses `matrix-profile: pr` to run a small representative matrix (Linux coverage/full-language, Linux latest Python, Windows, macOS). Release/hotfix/push validation must keep `matrix-profile: full`.
- `.github/workflows/test-coverage.yml` is manual-only because reusable-test already uploads coverage on the Linux coverage axis. Do not re-enable PR/push triggers unless reusable-test coverage is removed in the same change.
- `.github/workflows/ci.yml` owns ordinary PR routing through `scripts/ci_route.py` and `config/ci-routing.yml`. Expensive optional checks such as regression, SQL platform compatibility, benchmarks, and broad E2E must be path-routed or manual/scheduled instead of running unconditionally on every PR.
- For language-plugin changes, rely on `--change-impact --change-impact-scope ...` during development, run the focused command it reports, then let CI's single full-language axis provide the final cross-language golden gate.

## Agent Dogfood Feedback Loop

For non-trivial work, expert agents must use this project as their primary feedback instrument while they work, then preserve the learning in memory:

1. **Before edits:** run `uv run python -m tree_sitter_analyzer --change-impact --format json` to get the affected surface and verification command.
2. **During exploration:** prefer TSA queries over blind file scans. Use focused codegraph/query/health commands for the area being changed, and keep the raw command outputs small enough to compare before/after.
3. **After edits:** rerun change-impact and the reported verification command. For Python source changes, also run the local patch coverage gate from the Test Runtime Contract.
4. **Memory capture:** store a concise JSON record in project memory with `branch`, `task`, `tools_used`, `signals`, `decision`, `verification`, and `followups`. Use the available `memory_store` MCP when present; otherwise use the Claude Flow memory CLI (`npx @claude-flow/cli@latest memory store --namespace tsa/agent-feedback ...`). If neither memory backend is available, include the JSON in the final response so the lead can store it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aimasteracc/tree-sitter-analyzer](https://github.com/aimasteracc/tree-sitter-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
