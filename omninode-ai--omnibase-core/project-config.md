---
trigger: always_on
description: <!-- onex-allow-file-todo-marker reason="repository policy guide intentionally documents TODO marker examples" -->
---

<!-- onex-allow-file-todo-marker reason="repository policy guide intentionally documents TODO marker examples" -->

# CLAUDE.md - Omnibase Core

> **Python**: 3.12+ | **Framework**: ONEX Core | **Package Manager**: uv | **Shared Standards**: See **`~/.claude/CLAUDE.md`** for shared development standards (Python, Git, testing, architecture principles) and infrastructure configuration (PostgreSQL, Kafka/Redpanda, Docker networking, environment variables).

---

## Repo Invariants

These are non-negotiable architectural truths:

- **Nodes are thin** - Nodes are coordination shells, not business logic containers
- **Handlers own logic** - Business logic lives in handlers, not nodes
- **Reducers are pure** - `delta(state, event) -> (new_state, intents[])` with no I/O
- **Orchestrators emit, never return** - ORCHESTRATOR nodes cannot return `result`
- **Contracts are source of truth** - YAML contracts define behavior, not code
- **Unidirectional flow** - EFFECT → COMPUTE → REDUCER → ORCHESTRATOR, never backwards

---

## Agent Behavioral Rules

- Never disable pre-commit hooks, CI checks, or type checkers to make code pass. Fix the code instead.
- Never write state files to `~/.claude/` — use the workspace-local `.onex_state/` directory.

### Contract-first topic definitions

This repo defines the contract framework that all ONEX nodes use. Kafka topic
declarations belong in contract YAML files (`event_bus.publish_topics` /
`subscribe_topics`), not hardcoded in application code. Enforced by
`validation/validator_hardcoded_topics.py` plus the `demo-path-topic-coherence`
CI gate (`onex-demo-path-topic-gate`), and by
`validators/contract_config_compliance.py` (bare env reads, bus-bypass imports,
missing contract config; CI job `contract-config-compliance`).

---

## Non-Goals

We explicitly do **NOT** optimize for:

- **Backwards compatibility for internal APIs** - Internal Python module paths may change without notice. The stable external surface is defined in the External SDK Surface section below.
- **Convenience over correctness** - Contract violations fail loudly
- **Business logic in nodes** - Nodes coordinate; handlers compute
- **Dynamic runtime behavior** - All behavior must be contract-declared
- **Implicit state** - All state transitions are explicit and auditable
- **Tight coupling** - Protocol-based DI enforces loose coupling

---

## External SDK Surface

This package is consumed by external developers. The following surfaces are **stable**
and must not change without a deprecation path:

| Surface | Contract | Example |
|---------|----------|---------|
| `onex` CLI commands | Command names and flags are stable | `onex init`, `onex new node`, `onex validate`, `onex discover` |
| `onex.nodes` entry-point group | Group name is stable; external packages register nodes here | `[project.entry-points."onex.nodes"]` |
| Generated scaffold layout | Directory structure from `onex init` / `onex new node` is stable | `src/<pkg>/nodes/<name>/contract.yaml` |
| Contract validation | `onex validate` accepts any directory with valid contract.yaml files | No monorepo assumptions |

The following are **unstable** (may change without notice):
- Internal Python module paths (`omnibase_core.infrastructure.*`, `omnibase_core.services.*`, etc.)
- Node base class internal APIs
- Validation rule implementations (new rules may be added)

**CI enforcement**: `test_no_internal_deps.py` prevents internal OmniNode packages
from appearing in hard dependencies (`sdk-boundary-check` CI job).
`omnibase-compat` is the single exemption: compat is the shared substrate every
OmniNode repo is allowed to hard-depend on.

---

## Quick Reference

```bash
# Setup
uv sync --all-extras && pre-commit install

# Testing — PARALLEL BY DEFAULT: addopts pins -n4, --timeout=60, --reruns=2
uv run pytest tests/                    # All tests (4 xdist workers via addopts)
uv run pytest tests/ -n 0 -xvs          # Debug mode (disable parallelism)
uv run pytest tests/ --cov              # With coverage (fail_under=60 in pyproject)

# Code Quality
uv run mypy src/omnibase_core/          # Type checking (strict=true, 0 errors required)
uv run ruff check src/ tests/           # Linting
pre-commit run --all-files              # All hooks
```

**Test markers**: `--strict-markers` is on. Markers are registered in
`pyproject.toml` `[tool.pytest.ini_options] markers` plus `tests/conftest.py`
(`memory_intensive`, `isolated`) — read those two sources, not a hand-copied list.

**uv, not Poetry**: `uv sync --all-extras` / `uv run <command>` / `uv lock`. All
Python commands — including any spawned agent's — run via `uv run`, never bare
`python`/`pip`. Shared git/hook rules (`--no-verify` / `--no-gpg-sign`
prohibitions, no background commits) live in `~/.claude/CLAUDE.md`.

---

## Handler Output Constraints

| Node Kind | Allowed | Forbidden |
|-----------|---------|-----------|
| **ORCHESTRATOR** | `events[]`, `intents[]` | `projections[]`, `result` |
| **REDUCER** | `projections[]` | `events[]`, `intents[]`, `result` |
| **EFFECT** | `events[]` | `intents[]`, `projections[]`, `result` |
| **COMPUTE** | `result` (required) | `events[]`, `intents[]`, `projections[]` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OmniNode-ai/omnibase_core](https://github.com/OmniNode-ai/omnibase_core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
