---
trigger: always_on
description: Quick-reference for AI assistants working in the `weakincentives` repository.
---

# CLAUDE.md

Quick-reference for AI assistants working in the `weakincentives` repository.

______________________________________________________________________

## Core Philosophy

**The prompt is the agent.** Prompts are hierarchical documents where sections
bundle instructions and tools together. No separate tool registry; capabilities
live in the prompt definition.

**Event-driven state.** All mutations flow through pure reducers processing
typed events. State is immutable and inspectable via snapshots.

**Provider-agnostic.** Same agent definition works across OpenAI, LiteLLM, and
Claude Agent SDK via adapter abstraction.

______________________________________________________________________

## Guiding Principles

### Definition vs Harness

WINK separates what you own from what the runtime provides:

**Agent Definition (you own):** Prompt, Tools, Policies, Feedback

**Execution Harness (runtime-owned):** Planning loop, sandboxing, retries,
throttling, crash recovery, deadlines, budgets

The harness keeps changing; your agent definition should not. WINK makes the
definition a first-class artifact you can version, review, test, and port.

### Policies Over Workflows

**Prefer declarative policies over prescriptive workflows.**

A workflow encodes _how_ to accomplish a goal—a predetermined sequence that
fractures when encountering unexpected situations. A policy encodes _what_ the
goal requires—constraints the agent must satisfy while remaining free to find
any valid path.

| Aspect | Workflow | Policy |
|--------|----------|--------|
| Specifies | Steps to execute | Constraints to satisfy |
| On unexpected | Fails or branches | Agent reasons |
| Composability | Sequential coupling | Independent conjunction |
| Agent role | Executor | Reasoner |

**Key policy characteristics:** Declarative, Composable, Fail-closed, Observable

### Transactional Tools

Tool calls are atomic transactions. When a tool fails:

1. Session state rolls back to pre-call state
1. Filesystem changes revert
1. Error result returned to LLM with guidance

Failed tools don't leave partial state. This enables aggressive retry and
recovery strategies.

______________________________________________________________________

## MANDATORY: Definition of Done

**No work is considered complete until `make check` passes with zero errors.**

This is non-negotiable. Do not claim a task is complete, do not move on to the
next task, and do not commit until:

```bash
make check  # Must exit 0 with no errors
```

If `make check` fails: fix errors, re-run, repeat until all checks pass.

______________________________________________________________________

## Commands

```bash
uv sync && ./install-hooks.sh   # Setup - BOTH STEPS ARE MANDATORY

make format      # Ruff format (88-char lines)
make lint        # Ruff lint --preview
make typecheck   # ty + pyright (strict)
make test        # Pytest, 100% coverage required
make check       # ALL checks - MANDATORY before any commit
```

______________________________________________________________________

## MANDATORY: Git Hooks Installation

**Git hooks MUST be installed in every new development environment.**

```bash
./install-hooks.sh   # Run this after cloning or in any new environment
```

### Why This Matters

The pre-commit hook runs `CI=true make check`, which:

1. **Runs the FULL test suite** (not the testmon subset used for local iteration)
1. **Enforces 100% coverage** on all code paths
1. **Exactly emulates CI verification** that runs on pull requests

### The Problem Without Hooks

Without hooks, you might:

- Run `make check` locally (uses testmon, runs only affected tests)
- Commit code that passes local checks
- **Fail CI** because the full test suite reveals issues testmon skipped

### The Solution

The pre-commit hook automatically runs `CI=true make check` before every
commit, ensuring:

- ✅ What passes locally **will** pass in CI
- ✅ No surprises when your PR is reviewed
- ✅ Coverage gaps are caught immediately

**If hooks aren't installed, install them now:**

```bash
./install-hooks.sh
```

______________________________________________________________________

### Efficient Testing Workflow

`make check` and `make test` automatically detect local vs CI execution:

- **In CI:** Full test suite with 100% coverage enforcement
- **Locally:** Only tests affected by changes (uses testmon coverage database)

The first local run builds a coverage database (`.testmondata`). Subsequent
runs use this database to identify which tests cover changed code and skip the rest.
This dramatically reduces iteration time when working on focused changes.

## Architecture

```
src/weakincentives/
├── adapters/     # OpenAI, LiteLLM, Claude Agent SDK
├── contrib/      # Mailbox, optimizers
├── dbc/          # Design-by-contract decorators
├── evals/        # Evaluation framework
├── prompt/       # Section/Prompt composition
├── resources/    # Dependency injection
├── runtime/      # Session, events, lifecycle
├── serde/        # Dataclass serialization
└── ...
```

## Style Patterns

### Types & Dataclasses

- Strict pyright; annotations are source of truth—no redundant runtime guards
- Use `@dataclass(slots=True, frozen=True)` or `@FrozenDataclass()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weakincentives/weakincentives](https://github.com/weakincentives/weakincentives) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
