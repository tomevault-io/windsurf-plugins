---
trigger: always_on
description: - `macro_agents/`: Dagster project and AI agents (`src/macro_agents/`), tests in `macro_agents/tests/`.
---

# Repository Guidelines

## Project Structure & Module Organization
- `macro_agents/`: Dagster project and AI agents (`src/macro_agents/`), tests in `macro_agents/tests/`.
- `dbt_project/`: dbt models and configs (`models/` for staging, markets, analysis, etc.).
- `document_extraction/`: dbt-ml project for extracting SEC/FOMC documents into BigQuery.
- Root `makefile`: common dev, test, lint, and setup commands.

## Build, Test, and Development Commands
- `make setup-dagster`: install Dagster dependencies via `uv`.
- `make run-dagster`: start Dagster UI on `:3000`.
- `make test`: run Dagster tests via `pytest` (uses `uv`).
- `make ruff`: lint/format Python with Ruff.
- `make lint` / `make fix`: lint or fix dbt SQL with SQLFluff.
- `make typecheck-dagster`: run `ty` type-checker over `macro_agents`.
- `make dbt-deps`: explicitly install locked dbt packages (networked setup step).
- `make dbt-manifest`: parse dbt project.
- `cd document_extraction && uv sync`: install the locked dbt-ml environment.
- `cd document_extraction && uv run dbt-ml compile --target dev`: validate the extraction DAG without materializing data.

## Worktrunk Hooks (wt)
- Project hooks live in `.config/wt.toml`; user hooks live in `~/.config/worktrunk/config.toml`. Project hooks require first-run approval.
- Hook types include `pre-commit` (formatters/linters/type checks before the merge commit) and `pre-merge` (tests/build verification after rebase, before merge).
- Hooks run automatically during `wt merge`; manually run with `wt hook pre-commit`, `wt hook pre-merge`, or `wt hook show`. Use `--no-verify` to skip hooks or `--yes` to auto-approve.
- Recommended alias: add `alias wsc='wt switch --create --execute=claude'` to your shell config, then use `wsc <branch>` or `wsc <branch> -- 'task'` to create/switch a worktree, run hooks, and launch Claude with forwarded args.

## Git Worktrees (Recommended For Parallel Sessions)
- Use a dedicated worktree per Claude session to avoid clobbering edits in the same working directory.
- Create a new worktree from `main` using an issue-based branch name: `git worktree add -b feat/issue-123-signal-charts ../economic-data-project-full-wt-issue-123 main`
- List worktrees: `git worktree list`
- Remove a worktree when done (from the primary repo): `git worktree remove ../economic-data-project-full-wt-issue-123`

## Dagster `dg` Cheat Sheet
- `cd macro_agents` before running `dg` commands.
- Start local Dagster UI: `uv run dg dev`
- Validate definitions: `uv run dg check --module-name macro_agents.definitions`
- List jobs/assets: `uv run dg list`
- Launch a job: `uv run dg launch --module-name macro_agents.definitions --job dspy_analysis_job`
- Launch selected assets (comma-separated): `uv run dg launch --module-name macro_agents.definitions --job dspy_analysis_job --assets analyze_economy_state,generate_economy_state_charts`
- Launch with config file: `uv run dg launch --module-name macro_agents.definitions --job dspy_analysis_job --config /tmp/econ_state_charts.yaml`

## GitHub CLI (gh) Cheat Sheet
- Check auth: `gh auth status`
- List issues: `gh issue list`
- Create issue: `gh issue create`
- Create PR: `gh pr create`
- View PR: `gh pr view`
- List PRs: `gh pr list`
- Check PR status: `gh pr checks`
- Add PR comment: `gh pr comment`
- Checkout PR: `gh pr checkout <number>`

## Coding Style & Naming Conventions
- Python: 4-space indentation; use Ruff for formatting/linting (`make ruff`).
- dbt SQL: SQLFluff enforced style (`make lint`, `make fix`).
- Naming: keep module names and functions descriptive; follow existing patterns in each subproject.

## Testing Guidelines
- Framework: `pytest` for Python (`macro_agents`).
- Run Dagster tests: `cd macro_agents && uv run pytest tests/ -v`.
- Install dbt packages once with `make dbt-deps`; pytest performs one offline dbt parse per session.
- Network integration tests are opt-in: `RUN_NETWORK_TESTS=1 uv run pytest -m network`.
- Test standards:
  - Prefer behavior-focused assertions (status, metadata, materialized values) over snapshots.
  - Cover loading, empty, error, and populated states.
  - Use deterministic fixtures and local resources (temp DuckDB, ephemeral Dagster instances); avoid network calls.
  - Use `DagsterInstance.ephemeral()` and `build_op_context` as context managers so their database resources are disposed deterministically.
  - Keep tests isolated and fast; clean up temp files and avoid shared mutable state.

## Commit & Pull Request Guidelines
- Commit messages are typically short and imperative; many use Conventional Commit prefixes (`feat:`, `fix:`, `chore:`).
- Prefer small, focused commits with clear intent.
- PRs should include a concise summary, linked issues (if applicable), and screenshots for UI changes.
- Never merge PRs. Always leave merging to a human reviewer.
- Add a PR comment tagging `@codex` to request review immediately after opening every PR. Use `gh pr comment <PR_NUMBER> --body "@codex please review this PR."` and mention the review request in the handoff.

## Data Request Scoping (Wizard Loop 1)

A GitHub issue is dbt-workable if it involves models, tests, sources, metrics, schema,
SQL transformations, data quality, or freshness. When scoping a request:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [C00ldudeNoonan/economic-data-project](https://github.com/C00ldudeNoonan/economic-data-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
