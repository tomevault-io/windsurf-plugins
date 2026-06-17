---
trigger: always_on
description: General agent behavior lives in `.cursor/rules/karpathy-guidelines.mdc` (always applied).
---

# AGENTS.md

General agent behavior lives in `.cursor/rules/karpathy-guidelines.mdc` (always applied).
Python conventions live in `.cursor/rules/python-standards.mdc` (auto-applied to `*.py`).
This file documents project-specific context an agent can't grep for.

## Project Overview

NVIDIA AI Cloud Validation Suite - validation and management tools for NVIDIA ISV Lab GPU
cluster environments. Monorepo with three Python packages managed as a uv workspace:

- **isvctl** - CLI controller for cluster lifecycle (setup → test → teardown)
- **isvtest** - Validation framework engine (pytest-based with custom discovery)
- **isvreporter** - Test results reporter for ISV Lab Service API

## Common Commands

```bash
uv sync                # install workspace
make build             # build all packages
make test              # run tests
make demo-test         # run all my-isv configs end-to-end (ISVCTL_DEMO_MODE=1, ~10s, no cloud)
make lint              # ruff
make format            # ruff format
make plan              # render docs/test-plan.yaml to AsciiDoc + interactive HTML
uv run isvctl test run -f isvctl/configs/suites/k8s.yaml          # canonical invocation
uv run isvctl test run -f config.yaml -- -v -s -k "test_name"     # forward pytest args
```

## Step-Based Execution Model

The framework separates *doing* from *checking*:

```text
Config (YAML) → Script (any language) → JSON output → Validations (assertions)
```

1. Scripts (Python, Bash, ...) perform cloud operations and print structured JSON to stdout.
2. Validations are simple assertions over that JSON - no cloud SDK code in validations.
3. Validations reference step output via Jinja2: `"{{steps.create_network.vpc_id}}"`,
   `"{{region}}"`. The orchestrator warns when a template references a missing step or
   field (catches `ChainableUndefined` silent fallbacks).

### JSON contract discipline

- Test stdout JSON is the provider-neutral contract between scripts and
  validations. Use ISV-agnostic names and avoid AWS-specific resource concepts
  unless a validation or later step consumes them.
- Keep output minimal: `success`, `platform`, `test_name`, and
  `tests.<check>.passed/message/probes` are usually enough. Omit IDs, regions,
  endpoint inventories, and other fields that do not affect behavior.
- Failure/skip diagnostics are allowed, but keep them concise and generic:
  top-level `error`/`error_type`, `tests.<check>.error`, `skip_reason`, or
  `cleanup_errors`. Avoid raw provider responses and resource dumps.

### Lifecycle invariants (non-obvious)

- Phases run in order: `setup → test → teardown`.
- **Teardown runs after setup/test failures by default** so cloud resources get
  cleaned up - but it is skipped when `teardown_on_failure` is disabled, or when
  setup was requested in the same invocation but no setup steps actually ran.
- **Teardown is best-effort** - one failing teardown step does not block the others.
- **Standalone teardown** (`isvctl test run -f config.yaml --phase teardown`) runs
  unconditionally - useful after a previous run with `AWS_SKIP_TEARDOWN`.
- Multiple `-f` configs merge; later files override earlier ones.

## Architecture

### isvctl - orchestration

Entry point: `isvctl/src/isvctl/main.py` (Typer).

- `cli/` - subcommands (`test`, `deploy`, `clean`, `docs`, `report`)
- `orchestrator/` - `loop.py` (phase loop), `step_executor.py` (step + validation
  execution, supports `best_effort` mode), `commands.py` (timeouts), `context.py`
  (Jinja2 with missing-reference warnings)
- `config/` - `schema.py` (Pydantic), `output_schemas.py` (per-step JSON schemas),
  `merger.py` (multi-file merge)
- `remote/` - `ssh.py` (with jumphost), `archive.py`, `transfer.py` (SCP via jumphost proxy)
- `cleaner/` - resource cleanup

### isvtest - validation framework

Entry point: `isvtest/src/isvtest/main.py`.

`run_validations_via_pytest()` is the bridge isvctl calls. It transforms validation
configs to pytest format, runs native pytest, and returns rich in-memory results
(category, message) alongside the exit code.

- `core/validation.py` - `BaseValidation` abstract class
- `core/discovery.py` - finds `BaseValidation` subclasses and ReFrame tests
- `core/runners.py` - `LocalRunner`, `SlurmRunner`, ...
- `core/{k8s,slurm,nvidia,ngc,workload}.py` - domain helpers

Validation classes live in `isvtest/src/isvtest/validations/` grouped by domain
(`generic.py`, `cluster.py`, `instance.py`, `network.py`, `iam.py`, `security.py`,
`host.py`, `k8s_*.py`, `slurm_*.py`, `bm_*.py`). Each subclass declares
`labels: ClassVar[tuple[str, ...]]` for filtering and is auto-discovered.
`network.py` includes security group scoping checks for workloads, nodes, subnets,
and services.

Workloads (`isvtest/src/isvtest/workloads/`) are long-running tests (NIM, NCCL,
stress) labelled `("workload", "slow", ...)` with manifests and helper scripts
colocated.

Test config loaded from YAML/JSON via `config/loader.py`. Global fixtures in
`tests/conftest.py`. `tests/test_validations.py` dynamically generates pytest tests
from `BaseValidation` classes.

### isvreporter - results upload

Entry point: `isvreporter/src/isvreporter/main.py` (Typer).

- `client.py` - ISV Lab Service API client
- `auth.py` - OAuth2

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/ai-cloud-validation](https://github.com/NVIDIA/ai-cloud-validation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
