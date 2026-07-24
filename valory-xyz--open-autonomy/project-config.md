---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Open Autonomy is a Python framework for creating decentralized multi-agent systems (MAS) that run as autonomous services. Agents coordinate via an ABCI (Application Blockchain Interface) consensus mechanism backed by Tendermint, enabling trust-minimized, transparent operations including on-chain interactions.

Built on top of the **open-aea** (Autonomous Economic Agent) framework. The CLI entry point is `autonomy` (defined in `autonomy/cli/`).

## Common Commands

### Development Setup
```bash
make new_env && pipenv shell
autonomy packages sync --update-packages
pip install --no-deps plugins/aea-test-autonomy
```

### Testing
```bash
# Framework unit tests (autonomy/ and tests/) with coverage
make test

# Test a specific skill (e.g. skill=counter)
make test-skill skill=counter

# Test a specific subpackage with coverage
# tdir = test path (without "test_" prefix), dir = dotted module path
make test-sub-p tdir=connections/test_abci.py dir=connections.abci
make test-sub-p tdir=skills/test_counter/ dir=skills.counter

# Run a single test file directly
pytest -rfE tests/test_autonomy/test_cli/test_analyse.py

# Run tests for a specific package
aea test by-path packages/valory/skills/abstract_round_abci
```

**Docker images required for integration and e2e tests** — pure unit tests (no `-m integration` / `-m e2e`) need nothing; fixtures that do need docker are the ones that call out to `registries_scope_class`, `UseACNNode`, `UseFlaskTendermintNode`, `UseTendermint`, or `SlowFlaskTendermintDockerImage`:

```bash
# Pulled from Docker Hub:
docker pull valory/autonolas-registries:latest    # registries_scope_class (chain tests, e2e)
docker pull valory/acn-node:latest                # UseACNNode (register_reset e2e)
docker pull tendermint/tendermint:v0.34.19        # UseTendermint, test_runtime
docker pull valory/slow-tendermint-server:0.1.0   # register_reset recovery helpers

# Built locally from deployments/Dockerfiles/tendermint/ (the flask-wrapped tendermint):
TM="deployments/Dockerfiles/tendermint/"
docker build $TM -t valory/open-autonomy-tendermint:0.1.0 \
                 -t valory/open-autonomy-tendermint:1.0.0 \
                 -t valory/open-autonomy-tendermint:latest
```

`.github/workflows/main_workflow.yml` also pulls `valory/safe-contract-net` and `trufflesuite/ganache:beta`, but those fixture classes (`GnosisSafeNetDockerImage`, `GanacheDockerImage`) have no callers in this repo and can be omitted locally. `valory/contracts-amm` was previously in this pull list too; it was dropped after Docker Hub started returning 404 for the image and a confirm-no-callers check showed nothing in this repo invokes `AMMNetDockerImage`.

### Formatting & Linting
```bash
# Auto-format code
tomte format-code             # runs black + isort

# Check all code quality
tomte check-code              # runs all linters

# Individual linters via tox
tox -e black-check            # formatting check
tox -e isort-check            # import order check
tox -e flake8                 # style
tox -e mypy                   # type checking (strict, Python 3.14)
tox -e pylint                 # linting
tox -e darglint               # docstring validation (Sphinx style)
```

### Pre-commit routine (mandatory before every commit)

Run these in order — every commit. Skipping any one has caused CI failures
in the past (stale copyright years dirty the tree under `check-api-docs`,
skipped hash locks fail `common-checks-1`, etc.):

```bash
# 1. Auto-format (black + isort)
tomte format-code

# 2. Static analysis (black-check, isort-check, flake8, mypy, pylint, darglint)
tomte check-code

# 3. Security (bandit, safety, gitleaks)
make security

# 4. Bump copyright headers on every touched file (CI fails otherwise — it
#    runs `tomte format-copyright` as its FIRST step, then checks git diff)
tox -e fix-copyright

# 5. Dependencies — keep tox.ini / pyproject.toml / packages deps in sync
tox -e check-dependencies
```

**If you modified anything under `packages/`**, add:

```bash
# 6. Regenerate skill/agent artifacts + ABCI docstrings
make generators

# 7. Re-lock package hashes (MUST be last — any subsequent edit re-dirties
#    fingerprints and requires a re-lock)
autonomy packages lock

# 8. Keep autonomy/constants.py ABSTRACT_ROUND_ABCI_SKILL_WITH_HASH matching
#    the just-locked hash in packages.json
grep abstract_round_abci packages/packages.json | head -1
grep ABSTRACT_ROUND_ABCI_SKILL_WITH_HASH autonomy/constants.py

# 9. Update IPFS hashes in docs (package_list.md etc.)
tox -e fix-doc-hashes
```

**If you added or changed code docstrings**, also:

```bash
tox -e generate-api-documentation   # regenerate docs/api/**.md
```

**If you modified an `AbciApp` definition** specifically:

```bash
tox -e abci-docstrings
```

**Final verification before `git commit`**: run the CI-mirror checks so the
working tree is exactly what CI will see:

```bash
tox -e check-api-docs           # confirms docs/api/** matches current source
tox -e check-hash               # confirms packages.json matches
tox -e check-packages           # package-level validity
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [valory-xyz/open-autonomy](https://github.com/valory-xyz/open-autonomy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
