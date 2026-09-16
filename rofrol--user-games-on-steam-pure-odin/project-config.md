---
trigger: always_on
description: This file defines how humans and coding agents must develop and maintain this
---

# Architecture and Maintenance Policy

## Purpose

This file defines how humans and coding agents must develop and maintain this
system.

LLM agents can generate and modify code much faster than humans can review the
resulting architecture. An individual change may appear correct while gradually
increasing coupling, duplication, hidden assumptions, and the amount of context
required to understand the system.

As the codebase grows, an agent becomes less able to reason about it as a whole.
It may then solve local problems by adding adapters, abstractions, compatibility
layers, special cases, and patches. Each addition makes the next change harder
and accelerates architectural decay.

Complexity is therefore a limited resource. The goal is not to maximize the
amount of code produced. The goal is to keep the system small, coherent,
replaceable, and understandable by both humans and agents.

These rules exist to guide judgment, not to replace it with arbitrary metrics
or procedural paperwork.

## Human Responsibility

Humans retain control of the architectural core. A human must approve changes
to:

- system-wide architecture and module boundaries;
- core domain models and invariants;
- public interfaces and persistent data formats;
- measurable product and operational requirements;
- security and trust boundaries;
- decisions that would be expensive or difficult to reverse;
- rewrites that replace existing modules.

Agents may propose such changes, explain their trade-offs, and prepare a plan,
but must not silently make them.

## Classifying Changes

A change is non-trivial if it does any of the following:

- materially alters observable behavior, a public contract, or persistent
  data;
- adds or changes a dependency, module boundary, or trust boundary;
- changes concurrency, authorization, security, or failure behavior;
- introduces a new architectural concept or assumption;
- requires coordinated edits in places that can no longer be understood
  independently;
- cannot be verified safely through local reasoning and focused tests.

A change may still be trivial when it makes a small, local, easily reversible
adjustment to presentation or other non-contractual behavior, provided its
consequences can be established locally and verified directly. Otherwise, a
change is trivial only when behavior and contracts remain unchanged, no
boundary or architectural assumption changes, correctness can be established
locally, and the change is easy to reverse.

Classify changes by their consequences, not by line count, file count, function
length, or similar proxy metrics. When uncertainty about material consequences
remains after proportionate inspection, treat the change as non-trivial.

A consequence is material when it changes something a user, caller, operator,
or neighboring module may reasonably rely on; requires migration or
coordinated changes; expands authority or failure impact; introduces a durable
constraint; or would be costly to reverse.

## Rules for Agents

### 1. Protect the Architectural Core

- Keep the architectural core as small as possible.
- Prefer moving optional behavior to replaceable modules around the core.
- Do not expand the core merely to make a local implementation easier.
- Ask for human approval before changing the core or its fundamental
  assumptions.

### 2. Enforce Module Boundaries

- Draw boundaries around cohesive responsibilities and design decisions that
  may change independently, not around process steps, technical fashions, or
  arbitrary layers.
- Prefer deep modules: narrow interfaces that hide substantial complexity and
  expose the smallest useful contract. Do not create shallow wrappers whose
  interfaces expose nearly as much complexity as their implementations.
- A boundary should hide knowledge that other modules do not need. If callers
  must understand the implementation, duplicate its assumptions, or coordinate
  with its internal changes, the boundary is misplaced or incomplete.
- Define narrow, explicit interfaces between modules.
- Enforce boundaries mechanically with the language, type system, build system,
  dependency rules, linters, or architecture tests.
- Do not bypass a boundary through internal imports, shared mutable state,
  database access, reflection, or duplicated implementation knowledge.
- Treat a boundary that exists only in documentation as incomplete.

### 3. Protect Trust Boundaries

Treat the following as trust-boundary crossings unless the project explicitly
defines otherwise:

- user-controlled or externally supplied input;
- network, filesystem, subprocess, and device I/O;
- databases and other persistent state;
- secrets, credentials, and authentication material;
- third-party code, services, and dependencies;
- privileged operations and externally visible side effects.

At each trust boundary:

- validate and normalize untrusted data before it enters the core;
- keep authentication, authorization, and side effects explicit;
- use narrow representations that express what has already been validated;
- do not allow raw untrusted representations, secrets, or ambient authority to
  spread through the system;
- ask for human approval before creating, removing, or materially changing the
  boundary.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rofrol/user-games-on-steam-pure-odin](https://github.com/rofrol/user-games-on-steam-pure-odin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
