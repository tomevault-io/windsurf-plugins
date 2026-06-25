---
trigger: always_on
description: - `Constraint-Driven Evolution`: Add structure when real project constraints justify it.
---

# Project Context

## Meta-Protocol Principles

- `Constraint-Driven Evolution`: Add structure when real project constraints justify it.
- `Single Source of Truth`: Keep durable protocol, open work, completed delivery, and docs in separate files.
- `Context Hygiene`: Compress stale context before it becomes coordination drag.
- `Boundary Clarity`: README is the human entrypoint, `AGENTS.md` is durable protocol, `BACKLOG.md` is open work, and `CHANGELOG.md` is delivery history.

## Concept

`pi-actors` is a local-first actor runtime and orchestrator for Pi. It wraps trusted local programs, scripts, services, pipelines, and recipes as addressable actors that agents can `spawn`, control with typed `message` envelopes, and observe with `inspect`. It also persists user/agent-registered actor-control tools as recipe files under `~/.pi/agent/recipes`, giving agents durable operational muscle memory for launching and managing the local actor zoo.

Treat this extension as an experimental self-evolution membrane for the agent harness: a way for agents that are not pretrained on local workflows to acquire, preserve, inspect, and refine operational capabilities through explicit local actors, recipes, fixtures, skills, and state rather than hidden assumptions. Keep that potential grounded in small, testable, operator-visible protocol slices.

## Topology

```text
Pi host
  -> index.ts composition root
     -> lib/tools*.ts / prompts.ts      public tool + injected prompt surface
     -> lib/runtime.ts / registry.ts     active user recipe tools
     -> lib/recipes-*.ts                 packaged/user/draft recipe discovery
     -> lib/async-runs.ts                spawn lifecycle and run state
     -> lib/rooms.ts               room, roster, mailbox, communication log
     -> scripts/*.mjs                    self-contained process entrypoints
     -> recipes/*.json                   packaged actor components
     -> skills/* + docs/*                agent guidance and transportable specs
```

- `/index.ts`: Minimal extension coordinator/composition root. It wires live pi ports and should avoid owning domain behavior.

## Domain Modules

- `/lib/*.ts`: Flat Domain DAG modules for cohesive reusable behavior.
  - `command-templates.ts`: portable command-template execution graph.
  - `tools-access.ts`: shared public tool access/session ownership guards and normalized mismatch diagnostics.
  - `tools-mailbox.ts`: mailbox contract normalization and accepted/emitted message type helpers for public tool responses.
  - `schema.ts`: tool arg declarations and placeholder-derived schemas.
  - `identity.ts`, `paths.ts`, `config.ts`: names, paths, and persistence.
  - `registry.ts`, `runtime.ts`: register/update/delete, load/conflict/registration coordination.
  - `execution.ts`, `execution-output.ts`, `limits.ts`: registered-tool execution and bounded output.
  - `recipes-references.ts`, `recipes-discovery.ts`, `recipes-usage.ts`: recipe graph, discovery, and usage metadata.
  - `async-runs.ts`: detached run lifecycle facade; `runs-*` subdomains own artifacts, start guards, status, indexes, inbox/outbox, delivery, process control, and retention internals.
  - `runtime-notifier.ts`, `mailbox-loop.ts`: wake notifications and reusable run/branch mailbox worker loops.
  - `messages.ts`, `rooms.ts`, `recipes-context.ts`, `inspector.ts`, `observability.ts`: addressed message protocol, rooms, recipe prompt context, communication previews, and ambient run status.
  - `prompts.ts`, `temp.ts`: LLM-facing copy and temp cleanup.
  - `tools.ts`: public tool family composition and reserved tool names.
  - `tools-message.ts`: public `message` tool behavior, including run controls, branch/room routing, tool actor invocation, and delivery feedback.
  - `tools-inspect.ts`: public `inspect` tool behavior, including recipe registry, room/session/tool/run views, and observation formatting.
  - `tools-spawn.ts`: public `spawn` tool behavior, including actor launch, draft recipe capture, and launch diagnostics.
  - `tools-local.ts`: saved local capability execution, generated schemas, value normalization, and async recipe launch.
  - `tools-register.ts`: public `register_tool` behavior and schema for persisted local capability registration.
  - `tools-response.ts`: compact model-facing responses and next-action rendering shared by public tool execution paths.

## Repo Surfaces

- `/scripts/*.mjs`: Stable executables for detached/helper processes. Prefer self-contained script ownership; do not preemptively move script logic into `lib/` just to make scripts thin.
- `/lib/*.ts`: Compiled reusable domains for the extension/runtime. Move script behavior into `lib/` only when it has real non-script consumers or belongs to an existing reusable domain. Packaged JS-only execution, tests, or shim neatness alone do not justify a new `lib/` domain.
- `/recipes/*.json`: Packaged standard recipe library. Keep recipes optional, composable, policy-light, and caller-configurable.
- `/skills/actors/SKILL.md`: Dense practical reference for operating pi-actors itself.
- `/skills/swarm/SKILL.md`: Bundled methodology skill for multi-agent standards, strategies, and portable examples.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [llblab/pi-actors](https://github.com/llblab/pi-actors) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
