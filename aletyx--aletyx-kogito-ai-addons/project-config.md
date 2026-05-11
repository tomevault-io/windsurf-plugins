---
trigger: always_on
description: Orientation for AI coding agents (and humans) modifying this repo. Skim
---

# Agent guide

Orientation for AI coding agents (and humans) modifying this repo. Skim
this before changing code; it captures decisions that aren't obvious from
reading the source.

> Filename note: this follows the [agents.md](https://agents.md)
> convention. Tools like Cursor, Aider, Claude Code, and Codex
> auto-discover `AGENTS.md` at the repo root.

## What this is

The repo ships a family of GenAI add-ons for [Apache KIE](https://kie.apache.org/),
targeting its Kogito cloud-native runtime. Today there's one add-on — the **Ad-Hoc Subprocess**
orchestrator — that lets an LLM drive a BPMN ad-hoc subprocess: pick the
next task, write a prompt, parse structured output, write results back to
process variables, and signal completion.

Multi-runtime is core to the design: the same orchestrator runs on
**Quarkus** (LangChain4j) and **Spring Boot** (Spring AI). The core
module is framework-agnostic.

## Module layout (one rule)

Every artifact follows the same prefix. Once you know the rule, the
layout is predictable:

```
groupId:    ai.aletyx.kogito
artifactId: aletyx-kogito-ai-{addons-{runtime}-?}{role}{-suffix?}
package:    ai.aletyx.kogito.addons.{adhocsubprocess|quarkus|springboot}.…
```

`addons-{runtime}-` infix appears only on runtime-specific modules.
The framework-agnostic core and the framework-agnostic JPA piece don't
carry it (matches Apache Kogito's `kogito-addons-quarkus-*` convention).

```
aletyx-kogito-ai-parent (root)
├── aletyx-kogito-ai-bom                          ← BOM for consumers
├── aletyx-kogito-ai-build-parent                 ← shared dep + plugin mgmt
└── aletyx-kogito-ai-adhoc-subprocess-parent      ← feature aggregator
    ├── aletyx-kogito-ai-adhoc-subprocess         ← framework-agnostic core
    ├── aletyx-kogito-ai-adhoc-subprocess-storage-jpa
    ├── aletyx-kogito-ai-addons-quarkus-adhoc-subprocess
    ├── aletyx-kogito-ai-addons-quarkus-adhoc-subprocess-storage-jpa
    ├── aletyx-kogito-ai-addons-springboot-adhoc-subprocess
    └── aletyx-kogito-ai-addons-springboot-adhoc-subprocess-storage-jpa
```

## Architecture in 5 bullets

- **Core is plain Java.** No `@ApplicationScoped`, no `@Component`. The
  runtime addons construct core classes via their respective producers.
  Don't add CDI / Spring annotations to anything in
  `aletyx-kogito-ai-adhoc-subprocess`.
- **Two SPIs gate the boundaries.** `LLMProvider` is the single
  LLM abstraction (one bean, three methods: `selectTasks`,
  `formulatePrompt`, `executeTask`). `AIContextRepository` is the storage
  abstraction. Consumers can swap either by registering their own bean.
- **Three-pass orchestration.** Selection → formulation → execution.
  Each call has a narrow goal; we don't ask the LLM to "do everything"
  in one prompt. See `AIIntelligence`.
- **Phase state machine.** `PRE_HUMAN → AI_PROCESSING → POST_HUMAN →
  COMPLETED`, terminal at `COMPLETED`. Mutate via
  `AIContext.transitionTo(Phase)`, never via `setCurrentPhase` (that
  exists only for repository round-trips).
- **The orchestrator never embeds an HTTP client.** Provider-specific
  bits (LangChain4j, Spring AI) live in the runtime add-ons under
  `…/llm/`. Don't reach for an SDK from the core module.

## Decisions worth knowing

- **Per-runtime tools are intentional.** LangChain4j `@Tool` (Quarkus)
  and Spring AI `@Tool` (Spring Boot) aren't unified behind a portability
  layer. The cost — duplicating tool annotations when migrating runtimes
  — is acknowledged. Don't propose a tool-abstraction layer; it's been
  considered and rejected.
- **We use plain LangChain4j, not the Quarkus extension.**
  `quarkus-langchain4j` brings a REST client stack that conflicts with
  our resteasy setup. The producer in `LLMProviderProducer` builds
  `AnthropicChatModel` / `OpenAiChatModel` / `OllamaChatModel` directly.
- **`max_completion_tokens` not `max_tokens`** for OpenAI. Older models
  accept both; gpt-5.x and o-family reject `max_tokens`. We always send
  the new form.
- **kie-flyway, not the runtime's default Flyway.** Migrations ship in
  the storage add-on jar at
  `kie-flyway/db/aletyx-kogito-ai-adhoc-subprocess/`. On Quarkus this
  needs `quarkus.flyway.migrate-at-start=false` and `kie.flyway.enabled=true`
  to avoid two runners fighting.
- **`ResourceLoader.load()` caches.** Skill / BPMN / task schemas are
  immutable per JVM, and we read each on every evaluation tick. The
  cache is intentional. JVM restart is the only invalidation path
  (a `clearCache()` test hook exists, but production code shouldn't
  call it).
- **Startup validator fails fast.** A typo in the provider name or a
  missing API key fails the application boot via
  `LLMProviderValidator.validate()` — not the first chat call. See
  `LLMProviderStartupValidator` in each runtime addon.
- **REST endpoints are opt-out**, not opt-in. The
  `/api/ai-history/*` controllers register by default and can be
  disabled via `aletyx.kogito.ai.adhocsubprocess.history-api.enabled=false`.

## Property + table conventions

```
property prefix: aletyx.kogito.ai.adhocsubprocess.…
table prefix:    aletyx_kogito_ai_…
```

Both runtimes share the same property tree — consumers pin keys once
and switching runtimes doesn't move them. JPA tables are prefixed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aletyx/aletyx-kogito-ai-addons](https://github.com/aletyx/aletyx-kogito-ai-addons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
