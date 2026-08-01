---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The **Trader** repo hosts the Olas prediction-market agents. A single agent package (`valory/trader`) is shipped as two services:

| Service | Stack name | Chain | Venue |
|---|---|---|---|
| `valory/trader_pearl` | **Omenstrat** | Gnosis | [Omen](https://aiomen.eth.limo/) |
| `valory/polymarket_trader` | **Polystrat** | Polygon | [Polymarket](https://polymarket.com/) (CLOB v2) |

Both run as **single-agent (sovereign) deployments** distributed via Pearl. The agent queries an AI Mech for probability estimates, evaluates profitability, and executes on-chain via a Safe multisig. Built on [Open Autonomy](https://stack.olas.network/) (ABCI skills, FSMs, content-addressed packages); Tendermint is framework plumbing, not the deployment shape.

When working locally with `make run-agent` / `aea-helpers run-agent`, the agent-level [`aea-config.yaml`](./packages/valory/agents/trader/aea-config.yaml) defaults are **Omen-flavored** — service-level `polymarket_trader/service.yaml` overrides only apply under `autonomy deploy`. Local Polystrat dev requires explicit overrides (`IS_RUNNING_ON_POLYMARKET=true`, `MECH_CHAIN_ID=polygon`, `DEFAULT_CHAIN_ID=polygon`, pUSD-scaled `STRATEGIES_KWARGS`, the Polymarket `TOOLS_ACCURACY_HASH`, and the chain-specific market-filter flags). See `README.md` for the full set.

## Tech Stack

- **Framework**: Open Autonomy
- **Package management**: `uv` (versions pinned in `pyproject.toml` — check there for the current Python range and dependencies; do not duplicate them here)
- **Task running**: `Makefile` + `tox`
- **Lint / format**: `tomte` (wraps black, isort, flake8, mypy, pylint, darglint, bandit)
- **Tests**: `pytest` + `hypothesis`

## Common Commands

### Testing
```bash
# Run all skill tests (pick the env that matches your interpreter; 3.10–3.14 supported)
uv run tox -e py3.10-linux     # or py3.11/3.12/3.13/3.14-linux, *-darwin, etc.

# Run a single skill's tests
uv run pytest packages/valory/skills/<skill_name>/tests/ -v

# Run a specific test
uv run pytest packages/valory/skills/<skill_name>/tests/test_behaviours.py::TestClassName::test_method -v
```

### Linting & Formatting
```bash
make format              # Auto-format (black + isort via tomte)
make code-checks         # All linting: black, isort, flake8, mypy, pylint, darglint
make security            # bandit + safety + gitleaks
make common-checks-1     # copyright, dependencies, linting
make common-checks-2     # hash check, package check, ABCI checks
make all-checks          # Everything
make ci-linter-checks    # CI linter checks (the full CI lint suite)
```

### Code Generation & Hashes
```bash
make generators          # Update hashes, copyright headers, ABCI docstrings
make sync-packages       # Sync package versions across the repo
# Update FSM specs for a specific skill
autonomy analyse fsm-specs --update --app-class <AppClass> --package packages/valory/skills/<skill_name>
```

### Running
```bash
# Local single-agent dev loop (wraps `uv run aea-helpers run-agent --name valory/trader --connection-key`)
make run-agent

# Containerized service deployment (pick the flavor)
uv run autonomy fetch --local --service valory/trader_pearl       # Omenstrat
uv run autonomy fetch --local --service valory/polymarket_trader  # Polystrat
```

## Project Structure

After `autonomy packages sync`, the layout looks like:

```
packages/valory/
├── agents/trader/                      # The single agent definition (used by both services)
├── connections/                        # polymarket_client (Polystrat CLOB v2), genai, x402, http_*, ipfs, ledger, ...
├── contracts/                          # Smart contract interfaces (FPMM, Conditional Tokens, Realitio, Safe, ERC-20/1155, ...)
├── customs/                            # Pluggable bet-sizing strategies: fixed_bet, kelly_criterion
├── services/
│   ├── trader_pearl/                   # Omenstrat service (Gnosis / Omen)
│   └── polymarket_trader/              # Polystrat service (Polygon / Polymarket CLOB v2)
└── skills/
    ├── trader_abci/                    # Main orchestrator / composed app
    ├── decision_maker_abci/            # Bet evaluation + placement (largest skill)
    ├── market_manager_abci/            # Market discovery (Omen + Polymarket variants)
    ├── mech_interact_abci/             # Mech communication
    ├── staking_abci/                   # Staking management
    ├── tx_settlement_multiplexer_abci/ # Routes settlement transactions
    ├── check_stop_trading_abci/        # Pause/stop conditions
    ├── agent_performance_summary_abci/ # Performance + payout tracking
    ├── chatui_abci/                    # Web UI hooks
    └── funds_manager/                  # Funds bookkeeping (uses RPC_URLS)
```

## Architecture

### ABCI Skill Pattern (core abstraction)

Every skill follows the Open Autonomy ABCI pattern — a finite state machine (FSM) replicated across agents via Tendermint consensus. Each skill contains:

- **`rounds.py`** — State (Round) classes defining consensus logic and transitions
- **`behaviours.py`** — Behaviours that execute at each FSM state (one per round)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [valory-xyz/trader](https://github.com/valory-xyz/trader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
