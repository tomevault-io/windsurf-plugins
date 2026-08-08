---
trigger: always_on
description: This file defines how humans and coding agents must design, implement, test, and
---

# Engineering and Architecture Guide

## Purpose and Scope

This file defines how humans and coding agents must design, implement, test, and
review this repository. It applies to the entire repository unless a more
specific `AGENTS.md` exists in a subdirectory.

The architectural source of truth is
[`SAP_AI_Agent_Platform_OpenHands_Implementation_Plan.md`](SAP_AI_Agent_Platform_OpenHands_Implementation_Plan.md).
Read the relevant sections of that document before making architectural,
security, runtime, workflow, SAP integration, or data-model changes.

This guide turns that product and architecture plan into day-to-day engineering
rules. It must not be used to weaken a boundary or control from the reference
plan. When requirements conflict or an important decision is not documented,
stop, state the conflict, and create or propose an Architecture Decision Record
(ADR).

## Default Engineering Rules

- Use **Python 3.13** for backend and platform code.
- Use **TypeScript in strict mode** only for frontend code.
- Choose the simplest solution that fully meets the current requirement.
- Do not add speculative abstractions, frameworks, services, or dependencies.
- Keep functions and modules small, readable, typed, and easy to test.
- Give every module one clear responsibility.
- Separate business rules from APIs, databases, frameworks, and vendor SDKs.
- Depend on interfaces at system boundaries; keep implementations replaceable.
- Do not let one component perform validation, authorization, execution,
  approval, and publication as a single responsibility.
- Reuse existing project patterns before creating new ones.
- Add tests for new behavior and bug fixes.
- Run Ruff, Pyright, and pytest before considering work complete.

Use these dependency boundaries:

```text
API and external adapters -> application use cases -> domain
```

The domain contains business rules. The application layer coordinates use
cases. Ports define required interfaces. Adapters connect databases, APIs, SAP,
OpenHands, Restate, and other external systems. The API layer only handles
transport concerns.

## Product Direction

Build a configuration-driven SAP AI agent platform in which:

- OpenHands is the execution kernel for agent reasoning, conversations, tools,
  context management, and isolated workspace operations.
- The product control plane owns tenant configuration, identity, authorization,
  run records, approvals, artifacts, and audit indexing.
- A durable workflow engine owns long-running phase state, retries, timers,
  human waits, cancellation, recovery, and coordination of external effects.
- The Runtime Manifest Builder deterministically resolves and freezes all
  approved runtime configuration before execution begins.
- The SAP MCP Gateway is the only route from agent execution to SAP systems.
- SAP remains the system of record for business data.
- Product behavior varies through versioned configuration, process packs,
  skills, capability providers, policies, and templates—not through
  customer-specific branches in the shared runtime.

Complete one production-quality vertical slice before expanding to additional
use cases. Prefer an end-to-end path with real validation over several partially
implemented components.

## Technology and Language Baseline

Use **Python 3.13** as the primary backend language. The control plane, Runtime
Manifest Builder, configuration resolver, durable workflow handlers, Runtime
Manager, OpenHands integration, SAP MCP Gateway, capability providers, artifact
service, and audit service should be implemented in Python unless an approved
ADR establishes a concrete reason to use another language.

Use **TypeScript in strict mode** for the web console and browser-side code.
Do not add a second backend implementation in TypeScript merely to share types
with the frontend. Exchange versioned OpenAPI, JSON Schema, and event contracts
and generate language-specific clients or models where practical.

Use YAML or JSON only for declarative, versioned configuration. Configuration
must select registered implementations and must not become an embedded
programming language.

The initial Python toolchain should use:

- `uv` for Python version, dependency, lockfile, and workspace management.
- Pydantic models for validated external and persisted contracts.
- FastAPI for REST, SSE, health, and administration endpoints.
- The Restate Python SDK for durable workflow handlers.
- `pytest` for tests, Ruff for formatting and linting, and Pyright for static
  type checking.
- Full type annotations for application, domain, port, and public adapter code.

Pin the exact Python version and all dependency versions in repository-managed
files and runtime images. Keep framework-specific types at adapter boundaries.

## Simplicity, Modularity, and Separation of Duties

Write the simplest code that correctly satisfies the current requirement and
preserves the architecture. Simple means easy to read, test, change, and
operate—not compressed, clever, or incomplete.

### Keep code simple

- Prefer clear, direct code over clever patterns or unnecessary indirection.
- Implement only the current requirement. Do not build speculative extension

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ash9119i9/TrueAI-harness](https://github.com/ash9119i9/TrueAI-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
