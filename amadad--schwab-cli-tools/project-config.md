---
trigger: always_on
description: - Keep the CLI as the primary interface.
---

# Agent Guidelines

## Focus
- Keep the CLI as the primary interface.
- Maintain the JSON response envelope for agent consumption (`--json` output).
- Put shared portfolio logic in `src/core/portfolio_service.py`.
- Put shared market logic in `src/core/market_service.py`.
- Put portfolio context assembly in `src/core/context.py`.
- Use `PortfolioContext` from `src/core/context.py` as the canonical decision-context contract for recommendation generation and evaluation.
- Put shared context helper models in `src/core/context_models.py`.
- Put policy loading/evaluation in `src/core/policy.py`.
- Put shared JSON aliases/helpers in `src/core/json_types.py`.
- Put prompt templates in `src/core/prompts.py`.
- Put model-subprocess environment filtering in `src/core/model_subprocess.py`.
- Put recommendation-engine orchestration in `src/core/advisor_sidecar.py` and typed recommendation helpers in `src/core/advisor_*.py`.
- Put typed snapshot/account/market models in `src/core/models.py`.
- Put snapshot merge/normalization helpers in `src/core/snapshot_service.py`.
- Put portfolio-brief orchestration in `src/core/brief_service.py` with deterministic helpers in `src/core/brief_*.py`.
- Put shared brief pipeline types in `src/core/brief_types.py`.
- Keep CLI commands in `src/schwab_client/cli/commands/`.
- Use `context.py` for client access (cached singletons).
- Use `src/schwab_client/paths.py` for path/env resolution.
- Use `src/schwab_client/history.py` for canonical SQLite persistence and query views.
- Use `src/schwab_client/_advisor/` for recommendation-store persistence only.
- Use `src/schwab_client/_client/protocols.py` for shared raw Schwab transport protocols.
- Treat `docs/history.md` as the canonical reference for historical snapshot semantics.
- Keep experimental recommendation-engine work in the separate `schwab-advisor` path documented in `docs/advisor-sidecar.md` until it proves useful enough to merge into the main CLI/history flow.

## Architecture

```
src/schwab_client/cli/
├── __init__.py      # Package entrypoint + compatibility exports
├── parser.py        # argparse setup and aliases
├── router.py        # Command routing
├── context.py       # get_client(), get_cached_market_client()
├── output.py        # JSON envelope, formatters
└── commands/        # Lazy map + one file per command group
    ├── __init__.py  # Lazy handler map
    ├── portfolio.py
    ├── market.py
    ├── history.py
    ├── trade.py
    ├── admin.py
    ├── context_cmd.py
    ├── brief_cmd.py
    └── report.py

src/schwab_client/
├── advisor_cli.py   # Separate schwab-advisor recommendation-engine entrypoint
├── _advisor/        # Recommendation store schema + persistence
├── _client/         # Internal client mixins / shared helpers
├── _history/        # Internal history schema + normalization + store/mixins
├── history.py       # Public SQLite persistence + query surface
├── paths.py         # Centralized path/env resolution
├── auth_tokens.py   # Token paths, locking, metadata sidecar
├── secure_files.py  # Restrictive permissions for tokens/private DBs
└── snapshot.py      # Canonical snapshot collection
```

### Adding a New Command

1. Add handler function to appropriate `commands/*.py` file
2. Add the handler to the lazy map in `commands/__init__.py`
3. Add argparse subparser in `cli/parser.py`
4. Add routing in `cli/router.py`

For recommendation-engine-only flows, add subparsers in `src/schwab_client/advisor_cli.py`
instead of extending the main `schwab` parser.

### Key Patterns

- Use `get_client()` from `context.py` - never instantiate clients directly
- Prefer passing `PortfolioContext` objects through recommendation/brief orchestration instead of introducing new ad hoc context dict shapes
- Use `handle_cli_error()` from `output.py` for consistent error handling
- Use `print_json_response()` from `output.py` for JSON output
- Use `format_header()` from `output.py` for text section headers
- Keep public policy defaults in `config/policy.template.json`; use `private/policy.json`
  or `SCHWAB_POLICY_PATH` for local aliases and thresholds
- Reuse the managed token storage in `auth.py` / `auth_tokens.py` / `market_auth.py`; token JSON files
  are paired with a local SQLite `tokens.db` sidecar for locking and metadata

## Safety

- Never commit `.env`, `config/accounts.json`, `private/policy.json`, or anything under `private/` or `tokens/`.
- The SQLite history database belongs under `private/history/` and must stay local.
- The recommendation-engine DB belongs under `private/advisor/` and must stay local.
- Keep the repo matching upstream; put local data/artifacts in `private/`.
- Avoid hardcoding account numbers or API keys.
- Do not pass Schwab credentials, token paths, Resend keys, or brief-recipient env vars into model subprocesses; use `model_subprocess_env()`.
- Live trading is disabled by default. Never use `--live` or `SCHWAB_ALLOW_LIVE_TRADES` in automation.
- Prefer the root auth flow: `schwab auth`, `schwab auth login --portfolio`, `schwab auth login --market`.
- For headless/SSH auth, use `schwab auth login --portfolio --manual` or `schwab auth login --market --manual`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amadad/schwab-cli-tools](https://github.com/amadad/schwab-cli-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
