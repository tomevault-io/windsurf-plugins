---
trigger: always_on
description: BullX is a highly available, self-evolving AI Agent Operating System built on Elixir/OTP and PostgreSQL. The codebase is organized into six subsystems that boot under a single OTP supervision tree. PostgreSQL is the system of record for sessions, memory, and knowledge; process-local state is ephemeral and reconstructible on restart.
---

# BullX Agent Guidelines

BullX is a highly available, self-evolving AI Agent Operating System built on Elixir/OTP and PostgreSQL. The codebase is organized into six subsystems that boot under a single OTP supervision tree. PostgreSQL is the system of record for sessions, memory, and knowledge; process-local state is ephemeral and reconstructible on restart.

## The Zen of BullX Development

Do the task that was asked.

Do not silently change the task.

Correct is better than clever.

Consistent is better than complete.

Useful is better than theoretically perfect.

A deliberate tradeoff is not a bug.

A local preference is not an architecture finding.

Omissions matter.

Contradictions matter.

Ambiguities matter when they change behavior.

Mere disagreement is usually noise.

Reality is not smooth.

Production systems are negotiated with time, cost, failure, and change.

Code is not static.

Code grows, bends, splits, merges, and dies.

Design for the next change, not for a frozen diagram.

Simplicity is not shallowness.

Completeness is not always responsibility.

Edge cases have diminishing returns.

Complexity compounds.

Rot compounds faster.

Prefer deletion over addition.

Prefer reuse over invention.

Prefer boring contracts over clever machinery.

Prefer the chosen guarantee over an imagined stronger one.

Functional programming is not the goal of the Erlang VM.

It is a means to reliable, concurrent, inspectable systems.

Purity is useful when it protects a boundary.

Purity is harmful when it becomes ceremony.

Processes are fault boundaries, not nouns.

Supervision is architecture, not decoration.

If no failure boundary changed, do not move the supervision tree.

Before changing code, write the cleanup plan.

Before adding code, ask what can be deleted.

Before inventing a pattern, search for the existing one.

Before criticizing a tradeoff, check whether it was already settled.

If it was settled, inspect inside it.

Do not relitigate it.

Working drafts may think out loud.

Shareable documents must not.

Remove scaffolding, TODO theater, abandoned alternatives, and meta-writing before committing docs.

### Scope fidelity

When a document says a tradeoff is final, evaluate consistency inside that tradeoff.

Do not argue for theoretical completeness unless the user asked for it.
Do not optimize for a perfect static design. This system will change.

A design that handles every imagined edge case today may become the source of tomorrow's rot.

Prefer the smallest correction that preserves the chosen direction.

If something looks risky, first ask:
- Is it actually inconsistent with the stated goal?
- Is it an omission inside the chosen design?
- Is it a contradiction against another explicit decision?
- Or am I merely disagreeing with the tradeoff?

Only the first three are useful by default. The fourth is noise unless explicitly requested.

### Reality bias

Real systems are uneven.

Do not assume the cleanest theoretical model is the responsible one.

A little duplication may be better than a premature abstraction.

A manual recovery path may be better than a complex automatic one.

A weaker guarantee may be better than code that nobody can safely change.

Prefer designs that remain understandable after six months of patches.

Prefer code that can be deleted.

Prefer behavior that can be explained to an operator.

Prefer guarantees that the system can actually keep.

## Subsystems

BullX is a single `:bullx` OTP application. Subsystems are placed on one of two tiers inside that application, decided by independence/coupling:

- **L1 — nested inside core** (`lib/bullx/<x>/`, modules `BullX.<X>.*`): tightly coupled to core; not a first-class concern on its own.
- **L2 — top-level namespace peer of core** (`lib/bullx_<x>/`, modules `BullX<X>.*`): first-class architectural concern, weakly coupled to core. Follows the same pattern Phoenix uses for `lib/bullx_web/` + `BullXWeb.*` — same mix project, same OTP app, just a sibling namespace at the top of `lib/`.

Escalation only happens on observed pressure. L1→L2 requires that the module is a first-class concern (parallel to Gateway / Web / Brain / AIAgent), not a core subcomponent.

| Subsystem | Tier | Path | Top-level module |
|---|---|---|---|
| **Gateway** | L2 | `lib/bullx_gateway/` | `BullXGateway` |
| **Brain** | L2 | `lib/bullx_brain/` | `BullXBrain` |
| **AIAgent** | L2 | `lib/bullx_ai_agent/` (or `lib/bullx_ai_agent.ex` — currently a single namespace file until the jido_ai fork lands) | `BullXAIAgent` |
| **Web / control plane** | L2 | `lib/bullx_web/` | `BullXWeb` |
| **Runtime** | L1 | `lib/bullx/runtime/` | `BullX.Runtime` |
| **Skills** | L1 | `lib/bullx/skills/` | `BullX.Skills` |

Subsystem summaries:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentBull/bullx](https://github.com/AgentBull/bullx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
