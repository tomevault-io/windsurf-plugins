---
trigger: always_on
description: Agent operating manual for [trade/mark](https://github.com/trade/mark) (MARK Protocol). Plain Markdown; no special frontmatter.
---

# AGENTS.md

Agent operating manual for [trade/mark](https://github.com/trade/mark) (MARK Protocol). Plain Markdown; no special frontmatter.

## Purpose

This file tells coding agents how to run, verify, and change this repo safely. Human-facing narrative docs live in `README.md`, `CONTRIBUTING.md`, and `docs/`. Use this file for day-to-day agent work and routing.

Violating security, architecture, branch, or secret-handling rules here is PR-blocking.

## Precedence

When instructions conflict:

1. `docs/development/BRANCHING.md` - branch model, PR targets, CI policy, required checks, merge rules.
2. This file - agent workflow, security posture, tooling constraints, validation routing.
3. Domain runbooks - for example `contracts/AGENTS.md` and `contracts/ARCHITECTURE.md`.
4. `README.md` / `CONTRIBUTING.md` - contributor story and quick start.

If a higher-precedence document is stale against executable repo state, do not silently follow it. Verify against `package.json`, `.mise.toml`, `contracts/Makefile`, and `.github/workflows/`, then call out the drift.

Do not change the main `README.md` title (`# MARK`) or its opening project description and tagline unless the user explicitly requests it.

## Project snapshot

- What: ZK UTXO privacy pool and settlement infrastructure for EVM-compatible chains with Optimism Superchain support.
- Contracts: Foundry under `contracts/`; local chain via Anvil OP Sepolia fork; deployment and ops scripts under `contracts/script`.
- Circuits: Circom + circomspect under `circuits/`; package name `@mark/circuits`.
- Frontend: Vite + React at `http://localhost:5173`.
- Tooling: mise + uv + pnpm. Do not introduce `nvm`, bare `pip install`, `npm install -g`, Hardhat, or `foundryup`.

### Contract domains

Current contract source domains under `contracts/src`:

- `access`: shared access-control helper contracts.
- `bridge`: Superchain bridge adapter.
- `settlement`: settlement module and settlement verifiers.
- `pool`: ZK UTXO pool, public-input validation, fee policy, credit ledger, pool verifier.
- `withdraw`: burn-to-claim native withdrawal adapter.
- `token`: RYLA token primitives.
- `crypto`: Merkle tree, proof helpers, generated Poseidon adapter.
- `interfaces`: narrow cross-domain interfaces.
- `errors`: shared error types.

Keep `bridge`, `settlement`, `pool`, and `withdraw` isolated. Shared code belongs only in approved shared domains. Full rules live in `contracts/ARCHITECTURE.md`.

## Safe command execution

- Inspect changed task/config files before trusting repo-managed automation on unfamiliar branches, especially `.mise.toml`, `package.json`, `pnpm-lock.yaml`, `contracts/Makefile`, workflow files, and shell scripts.
- Prefer `mise exec -- <command>` when a one-off command needs the repo toolchain without changing shell state.
- `mise trust` is allowed only after checking `.mise.toml` for unexpected commands.
- Never print, commit, or paste private keys, real RPC URLs, deployer keys, attester keys, production env files, or GitHub tokens. Redact command output if needed.
- Use `uv` / `uvx` for Python tools. If a Makefile or external doc suggests `pip install`, translate it to the project-approved uv path unless the user explicitly asks otherwise.
- Do not run production deployment, release, or governance-mutating scripts with real secrets unless the user explicitly requests that operation.

## Setup

Check prerequisites without installing:

```bash
./scripts/bootstrap.sh --check
```

Standard setup after reviewing task files:

```bash
mise trust
mise install
pnpm install
```

Heavy first-time setup:

```bash
mise run setup
```

Fresh clones may need submodules. See `docs/operations/TROUBLESHOOTING.md`.

Foundry note: `.mise.toml` sets `FOUNDRY_PROFILE=ci` in repo shells. For interactive contract iteration, prefer:

```bash
cd contracts
FOUNDRY_PROFILE=default forge test --no-match-path 'test/{invariant,integration}/**'
```

## Development commands

`pnpm dev` runs `mprocs`, which requires a TTY. Do not rely on it from headless agents.

| Goal                        | Command                                                                               |
| --------------------------- | ------------------------------------------------------------------------------------- |
| Frontend only               | `pnpm dev:frontend` or `mise run frontend`                                            |
| Local Anvil OP Sepolia fork | `mise run anvil` in a separate terminal/background session; requires `OP_SEPOLIA_RPC` |
| Deploy to local Anvil       | `mise run deploy-local` after Anvil is up                                             |
| Full README supersim stack  | `pnpm dev:supersim` only when the user asks for that path                             |

## Validation routing

Pick the smallest command that validates the touched surface. Do not run broad gates for a tiny edit unless it is needed for confidence or the user asks for it.

| Change type                          | Required local validation before finishing                                                                                         |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trade/mark](https://github.com/trade/mark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
