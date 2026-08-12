---
trigger: always_on
description: enables `exactOptionalPropertyTypes` — plain `astro/tsconfigs/strict` does not
---

# CLAUDE.md

Rundown is a format for defining executable runbooks using Markdown.

## Packages

- `@rundown-org/parser` - Markdown runbook parser
- `@rundown-org/core` - Core runbook state management and execution
- `@rundown-org/cli` - Command-line interface (`rundown`, `rd`)
- `@rundown-org/mcp` - MCP server for AI agent integration
- `@rundown-org/claude-code-plugin` - Claude Code plugin for runbook
  orchestration

## Architectural Principles

These principles are foundational. They take precedence over local convenience
and are not negotiable on a per-PR basis.

**State machine drives Rundown logic.** All runbook behaviour — step
transitions, result aggregation, action dispatch, lifecycle — lives in the
XState state machine in `@rundown-org/core`. Other packages MUST invoke the
state machine; they MUST NOT re-implement, replicate, or work around its logic.
If a desired behaviour isn't expressible in the state machine today, extend the
state machine — don't add a shadow implementation elsewhere.

**The CLI is a thin wrapper.** `@rundown-org/cli` exposes the core state machine
to agents and humans. Its job is to invoke state transitions and observe their
output (events, diagnostics, exit codes). Runbook logic does not live in the
CLI. New CLI commands must dispatch into existing core APIs; they do not
introduce parallel execution paths, hidden state, or transition rules of their
own. The same constraint applies to `@rundown-org/mcp` and
`@rundown-org/claude-code-plugin` — they are alternate front ends to the same
core.

**Core values, in priority order.** When trade-offs arise, resolve them in this
order:

1. **Correctness** — the behaviour matches the spec and the runbook author's
   intent.
2. **Type safety** — invalid states are unrepresentable; types drive dispatch
   (see [Design Principles](#design-principles)).
3. **Clean architecture** — small, self-contained modules with clear seams
   between packages and within packages.
4. **Test coverage** — every behaviour-bearing change is pinned by tests at the
   right layer (unit, integration, property, mutation).

**Correctness over pragmatism.** Prefer making the work correct over shipping a
"pragmatic" shortcut that compromises the values above. A workaround that papers
over a state-machine gap, an `any` that hides a typing bug, a skipped test that
masks a regression, or a one-off branch in the CLI that should have been a core
capability — all are net-negative regardless of the time they save in the short
term. When in doubt, raise the design question rather than patching around it.

### Side-effect categorisation

When a side effect needs to happen during runbook execution, classify it into
one of three categories before deciding where the code lives. The category
determines the architectural pattern.

| Category | Description                                                                                                       | Pattern                                                                                                                  | Examples                                                                                                    |
| -------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| **A**    | Genuinely CLI. Inherently external to the runbook program.                                                        | Stays in CLI. CLI sends a typed event into the machine if state must update.                                             | stdin reads, terminal rendering, child-process `spawn` syscall, env-var reads, exit-code-to-process mapping |
| **B**    | Machine-owned. Logic is part of the runbook program. No external dependency.                                      | Machine invokes a `fromPromise` actor from core. Pure filesystem or pure computation.                                    | OUTPUTS capture, ARTIFACTS resolution, FOR iteration advancement, frontmatter `outputs:` storage            |
| **C**    | Machine-owned with DI callable. Logic is part of the runbook program, but execution requires an external service. | Machine invokes an actor parameterised by a callable supplied at machine-construction time. The callable is the DI seam. | Command execution (policy + spawn), helper invocation (helper registry)                                     |

Category B and C actors are placed under `packages/core/src/runbook/actors/`
(the directory is established by the first migration that needs it). Each actor
is a `fromPromise`-shaped function in its own file, takes a typed `input`,
returns a typed `output`, and does not know about the runbook state manager, the
actor service, or the CLI emitter. See
[docs/internal/architecture.md § Per-step substate pattern](docs/internal/architecture.md#per-step-substate-pattern)
for how the machine wires these actors into the per-step state graph.

A side effect that lives in the CLI but classifies as B or C is architectural

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobyhede/rundown](https://github.com/tobyhede/rundown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
