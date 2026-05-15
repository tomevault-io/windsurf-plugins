---
trigger: always_on
description: cekernel is a parallel agent infrastructure for Claude Code.
---

# cekernel Development Guide

cekernel is a parallel agent infrastructure for Claude Code.
It maps Unix concepts (processes, IPC, schedulers) onto Claude workflows.
See [README.md](./README.md) for architecture details.

## Principles

- When uncertain about Claude Code specifications or behavior, always consult primary sources (official documentation, GitHub issues) before answering. Do not guess.
- Links in CLAUDE.md are not references — they are part of the instructions. You MUST read linked documents and follow them.

## Design Decisions

When implementing something, always check existing patterns first.

- **Feasibility check before implementation**: When adopting an approach different from existing patterns, verify technical constraints first (e.g., tool availability, API limitations, call depth restrictions). Do not start implementation without confirming feasibility.
- **Document deviations in ADR**: When choosing not to use an existing pattern, record the reason in an ADR. See the [ADRs](#adrs) section for how to create one.

> **Background**: When designing the Reviewer, an existing spawn pattern was overlooked and a subagent-based approach was implemented instead, requiring a rework. The technical constraint (skill → agent → agent is not allowed) could have been discovered with prior investigation.

## Philosophy

cekernel's design is rooted in UNIX philosophy and TDD.

- [UNIX Philosophy](./docs/unix-philosophy.md) — Eric S. Raymond's 17 principles
- [TDD](./docs/tdd.md) — Red-Green-Refactor cycle and testing principles

These documents are symlinked in `.claude/rules/` for automatic loading by Claude Code.
This ensures Worker agents read the content without requiring explicit `Read` calls.

> **Note**: `.claude/rules/` symlinks do not work in git worktrees. Reviewers operating
> in worktrees should refer to the [Review](#review) section below, which extracts the
> essential review criteria from these documents.

## Review

Review criteria for Reviewer agents. This section exists in CLAUDE.md because
Reviewers operate in git worktrees where `.claude/rules/` symlinks are unavailable.
The criteria below are extracted from `unix-philosophy.md`, `tdd.md`, and
`claude-code-constraints.md`.

### Design Principles

Assess whether changes follow these UNIX philosophy principles:

- **Modularity**: Simple parts connected by clean interfaces. Each component has well-defined boundaries.
- **Clarity**: Readability over cleverness. Code serves future maintainers.
- **Simplicity**: No unnecessary complexity. Favor simple solutions over intricate ones.
- **Parsimony**: Avoid large programs unless clearly necessary.
- **Transparency**: Design for visibility. Systems should be immediately understandable.
- **Robustness**: Transparency and simplicity enable correctness verification.
- **Least Surprise**: Interfaces follow familiar conventions.
- **Repair**: Fail noisily and as soon as possible. Never silently continue in a broken state.
- **Separation**: Separate policy from mechanism; interfaces from engines.
- **Composition**: Programs accept and emit straightforward text streams for easy connection.

### Testing Criteria

- **Test behavior, not internals**: Tests verify externally observable behavior, not internal state.
- **Test independence**: Tests must not share state or depend on execution order.
- **Edge cases**: Cover null/empty values, boundary values (0, 1, max), and error paths.
- **TDD compliance**: When TDD is used, verify the Red-Green-Refactor cycle — failing test → minimal fix → refactor. Check commit suffixes: `(RED)`, `(GREEN)`, `(REFACTOR)`.
- **No tests for non-executable files**: If the change only modifies non-executable files (agent definitions, skill definitions, documentation), there should be no content-based tests (e.g., grep-testing `*.md` for specific strings). If such tests exist, request their removal.

### Platform Constraints

- **zsh compatibility**: Scripts `source`d in Claude Code must use `${BASH_SOURCE[0]:-${(%):-%x}}` fallback.
- **bash 3.2 compatibility**: No `declare -A` (associative arrays). Use temp files with `grep -qxF` instead.
- **Arithmetic safety**: Use `var=$((var + 1))` instead of `((var++))` (fails under `set -e` when var=0).
- **Subagent nesting**: Nesting depth ≥ 2 is unreliable. Prefer independent processes with FIFO IPC.
- **Context window**: Workers must externalize state to files/git — do not rely on conversation history.

## Scripts

### Basic Rules

All scripts must begin with:

```bash
set -euo pipefail
```

Source `shared/session-id.sh` to establish session scope:

```bash
source "${SCRIPT_DIR}/../shared/session-id.sh"
```

### Shared Helpers

Each helper in `scripts/shared/` has a header comment documenting its API (functions, arguments, return values). Read the script file directly for usage details.

### Known Pitfalls

`((var++))` returns exit 1 when `var=0` (bash treats 0 as falsy in arithmetic expressions).
Under `set -e` this causes immediate termination. Use `var=$((var + 1))` instead:

```bash
# BAD: terminates under set -e when FAILED=0
((FAILED++))

# OK
FAILED=$((FAILED + 1))
```

`declare -A` (associative arrays) requires bash 4+. macOS ships bash 3.2 by default,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clonable-eden/cekernel](https://github.com/clonable-eden/cekernel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
