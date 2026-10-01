---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`safe-eth-py` is a Python library (published to PyPI as `safe-eth-py`, previously `gnosis-py`), not a
service. It provides `EthereumClient`, a wrapper over `web3.py` with ERC20/ERC721/tracing/batching
helpers, the `Safe` contract classes and Safe transaction/signature handling, price oracles, clients
for external services (Etherscan, Blockscout, Sourcify, ENS, CowSwap, Safe Transaction Service) and
an optional Django layer.

It is a shared dependency of the Safe Python backends, so any public API change ripples into
`safe-transaction-service`, `safe-queue-service`, `safe-auth-service`, `safe-decoder-service` and
`safe-cli`. Check the consumers before renaming or changing the signature of anything exported.

## Team and Project Context

- **Team**: Platform
- **Repository**: `safe-global/safe-eth-py`

### Linear Guidelines

- Create issues under team **Platform** with the `eth-py` and `Backend` labels
- Use `add-new-address` for new chain address issues (that flow is automated, see below)
- Add PR links as issue attachments/links
- Branches follow the Linear name: `uxio/pla-<number>-<slug>`. Plain `feat/<scope>`, `fix/<scope>`,
  `chore/<scope>` branches are also used for work without an issue

## Development Setup

### Initial Setup
```bash
uv sync --group dev --all-extras --frozen
source .venv/bin/activate
pre-commit install -f
```

`--all-extras` pulls the `django` extra, which the test suite needs. `uv.lock` is the source of
truth: always sync `--frozen`, run `uv lock` after editing `pyproject.toml` and commit both.
`[tool.uv] exclude-newer = "7 days"` rejects packages published in the last 7 days, so a brand new
release cannot be locked yet.

### Running Tests

Tests need Postgres (Django test database) and a ganache node on `localhost:8545` started with the
fixed mnemonic (`-d`), because the test mixins deploy Safe and Multicall contracts on it:

```bash
docker compose up -d db ganache

# Run all tests
pytest

# Run a single test file
pytest safe_eth/safe/tests/test_safe.py

# Run a specific test
pytest safe_eth/safe/tests/test_safe.py::TestSafe::test_estimate_tx_gas

# Run with coverage
coverage run --source=safe_eth -m pytest -rxXs
coverage report

# compose up + pytest + compose down
./run_tests.sh
```

`DJANGO_SETTINGS_MODULE=config.settings.test` is set automatically by `pytest-env`
(`[tool.pytest_env]` in `pyproject.toml`), so plain `pytest` works. The `config/` package exists only
to run the Django part of the suite and is not shipped in the wheel.

### Linting and Type Checking
```bash
pre-commit run --all-files   # isort, black, flake8, mypy — this is what CI runs
mypy safe_eth
```

### Building the Docs
```bash
./build_docs.sh   # sphinx-apidoc + make html, output in docs/build
```

## Architecture

### `safe_eth.eth` — node access

`EthereumClient` wraps `web3.py` and composes domain managers, all built in its `__init__`:

- `.erc20` (`Erc20Manager`), `.erc721` (`Erc721Manager`), `.tracing` (`TracingManager`),
  `.batch_call_manager` (`BatchCallManager`) — all subclasses of `EthereumClientManager`
- `.multicall` (`Multicall`), deployed per chain or from `safe_eth/eth/multicall.py` addresses

New node-level features belong in a manager, not on the client itself. `async_ethereum_client.py`
mirrors the sync client for async callers; a feature added to one usually has to be added to both.

Performance rules that hold across the codebase:
- Prefer `batch_call` / multicall over N sequential RPC calls
- Prefer `fast_to_checksum_address` / `fast_is_checksum_address` / `fast_keccak`
  (`safe_eth/eth/utils.py`, pysha3-backed and `lru_cache`d) over the `Web3` equivalents

### Contracts are loaded from bundled ABIs

`safe_eth/eth/contracts/__init__.py` holds a `contracts` dict mapping a name to a JSON file under
`abis/`, and generates the `get_<name>_contract(w3, address)` functions dynamically with `setattr` at
import time. The module also declares typed stubs for those generated functions so mypy sees them.
Deployed-bytecode getters (`get_proxy_1_3_0_deployed_bytecode`, …) are `@cache`d and written by hand.

### `safe_eth.safe` — Safe protocol logic

`Safe.__new__` is a factory: `Safe(address, ethereum_client)` detects the deployed version over RPC
(or takes `version=` to skip the lookup) and returns the matching subclass from `_version_class_map()`
— `SafeV001`, `SafeV100`, `SafeV111`, `SafeV120`, `SafeV130`, `SafeV141`, `SafeV150`. Version specific
behaviour goes in the subclass, shared behaviour in `Safe`, and `SafeCompatibilityAdapter` holds what
1.4.1 and 1.5.0 share. `proxy_factory.py` and `compatibility_fallback_handler.py` use the same
version-subclass pattern.

Every contract wrapper extends `ContractBase` (`safe_eth/eth/contracts/contract_base.py`), which
requires a `get_contract_fn()` returning one of the generated contract getters and exposes a
`cached_property contract`.

Other core modules:
- `safe_tx.py`: builds, hashes (EIP-712) and signs Safe transactions
- `safe_signature.py`: parses the packed signature blob into `SafeSignature` / `SafeSignatureAsync`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [safe-global/safe-eth-py](https://github.com/safe-global/safe-eth-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
