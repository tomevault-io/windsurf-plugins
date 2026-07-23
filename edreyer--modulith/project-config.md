---
trigger: always_on
description: This repository is a Kotlin/Spring Boot modular monolith built around hexagonal architecture, CQRS, DDD, and functional programming practices.
---

# Project Guidelines

## Purpose
This repository is a Kotlin/Spring Boot modular monolith built around hexagonal architecture, CQRS, DDD, and functional programming practices.

The primary architectural rule is strict isolation:

- Each Maven module is an isolation boundary.
- A module may support one or more bounded contexts, but those boundaries must remain explicit.
- Boundaries are enforced at compile time through Maven artifacts and at runtime through separate Spring application contexts.
- Preserve that separation. Do not introduce shortcuts that couple one module's implementation to another module's implementation.

## Current Module Layout

The root build is Maven-based. Current modules include:

- `common`
- `server`
- `user`
- `user-api`
- `booking`
- `booking-api`
- `payment`
- `payment-api`

In the current structure, the `*-api` modules define contracts that other modules can depend on. The concrete bounded-context modules (`user`, `booking`, `payment`) depend on `common` plus their own API module. The `server` module composes the application.

## Architecture Rules

### Module Boundaries
- Treat every Maven module as a hard boundary.
- Do not add direct implementation dependencies from one bounded-context module to another.
- Cross-module communication should happen through published contracts, not through importing another module's internal implementation classes.
- If a new bounded context is needed, prefer a new Maven module rather than folding unrelated behavior into an existing one.

### Package Layout
Within a bounded context, follow this package structure:

- `domain`
- `application.workflows`
- `application.port.in`
- `application.port.out`
- `application.service` when orchestration or reusable application logic is needed
- `adapter.in.web`
- `adapter.out.persistence`
- `config`

Keep names aligned with the actual package structure already used in this repository. Do not collapse everything into generic `application` or `adapter` packages when a more specific subpackage already exists.

### Dependency Direction
- `domain` is the core and must not depend on `application`, `adapter`, or framework details.
- `application` coordinates use cases and depends on the domain plus ports.
- `application.workflows` must not depend directly on adapter implementations.
- `adapter.in` drives the application through `application.port.in`.
- `adapter.out` implements `application.port.out`.
- `config` wires the module together without leaking framework concerns into the domain.

These constraints are not aspirational only; the repository already enforces key parts of them with ArchUnit tests. Changes should preserve or strengthen those tests.

## Coding Style

### Kotlin Style
- Write idiomatic Kotlin, not Java written in Kotlin syntax.
- Prefer properties over trivial `get...()` accessors unless a framework interface requires a getter-style method name.
- Use descriptive names based on domain language.
- Prefer immutable data and narrow, specific types over primitive strings and numbers where business meaning exists.
- Prefer sealed interfaces/classes, exhaustive `when` expressions, and explicit variant types over open-ended hierarchies and flag-based modeling.

### Domain Modeling
- Model domain concepts with ADTs, sealed hierarchies, and value objects where appropriate.
- Keep domain objects immutable.
- Push invariants into types and constructors/factories so failures happen as early as possible.
- Prefer compile-time and validation guarantees over scattered runtime checks.
- Make illegal states unrepresentable whenever practical.
- Prefer refined domain primitives built with `@JvmInline value class` when they encode meaningful invariants without adding object overhead.
- Use private constructors plus companion factory methods or extension-based factories to ensure invalid instances cannot be created directly.
- Prefer non-nullability by design. Represent optionality, lifecycle, and state variants with types rather than nullable fields when the distinction is domain-significant.
- Encode valid state transitions in function signatures when possible so the compiler prevents invalid flows.
- Keep construction-time validation centralized near the type being created rather than scattered across workflows, controllers, and adapters.

### Functional Error Handling
- Use Arrow-based error handling patterns already present in the codebase.
- Prefer `Either`, `Raise`, `either {}`, and related helpers for business logic.
- Prefer typesafe error handling to exceptions in domain and workflow code.
- Avoid throwing exceptions in workflows and domain logic except at clear framework boundaries.
- Model failure modes explicitly as sealed error types or other closed algebraic hierarchies.
- Treat `context(_: Raise<...>)` requirements as part of a function's effective signature.
- Use `either {}` at the same architectural points where exception handling would otherwise be introduced, so error handling remains explicit and compile-time enforced.
- Translate low-level errors into higher-level domain or workflow errors at layer and module boundaries rather than leaking infrastructure-shaped errors upward.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edreyer/modulith](https://github.com/edreyer/modulith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
