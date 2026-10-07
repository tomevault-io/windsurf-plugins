---
trigger: always_on
description: <!-- SPECKIT START -->
---

# BrewUp — Copilot Instructions

<!-- SPECKIT START -->
## Active Spec Kit Feature

- **001-order-confirmation** — Sales Order Confirmation. Plan: [specs/001-order-confirmation/plan.md](../specs/001-order-confirmation/plan.md) · Spec: [specs/001-order-confirmation/spec.md](../specs/001-order-confirmation/spec.md)
<!-- SPECKIT END -->

## Solution Overview

BrewUp is a **DDD Modular Monolith** built on **.NET 10 / C#** using CQRS and Event Sourcing.
The framework is **Muflone** (`AggregateRoot`, `CommandHandlerAsync<T>`, `IntegrationEvent`, `DomainEvent`).
The message bus is **RabbitMQ** (via `Muflone.Transport.RabbitMQ`).
The write store is **EventStore** (gRPC); the read store is **MongoDB**.

## Spec-Driven Development Context

When working inside a Spec Kit flow, treat the specification, constitution, domain carrier, and architecture memory as durable context.

The model must not silently invent missing domain or architecture decisions.

If the feature introduces a new bounded context or domain authority, the implementation plan must decide explicitly whether that authority is:

1. an existing module;
2. a new BrewUp module to create;
3. an external system outside the repository;
4. out of scope for the current feature.

For the active feature, **Payment is a separate authority**.

If Payment behavior is implemented in this repository, Payment must be created as a full BrewUp module following the standard module structure.

### Key Rules

- **Never** put business logic in facades.
- **Never** put domain decisions in command handlers. Command handlers load aggregates, call aggregate methods, and save.
- **Never** reference another module's `Domain` or `Infrastructure` project.
- **Never** implement a bounded context as loose folders inside another module.
- **Never** introduce ad-hoc project names or ad-hoc folder structures.
- **Never** place Payment behavior inside Sales.
- **Never** place Warehouse stock mutation inside Sales.
- **Never** place module-specific commands or domain events in the wrong module.
- **Strongly-typed IDs** always derive from `Muflone.Core.DomainId(string value)`.
- **Enumeration pattern** (Smart Enum) for status/type values — derive from `BrewUp.Shared.Helpers.Enumeration`.
- **Test-First**: write a failing `CommandSpecification<T>` (Given/When/Expect) before implementing aggregate logic.
- Use `Guid.CreateVersion7()` everywhere, never `Guid.NewGuid()`.
- Use `ConfigureAwait(false)` on every `await` in non-test code.
---

## Architecture Rules

These architecture rules are authoritative for generated plans, tasks, and code.

They complement the bounded-context rules in the feature specification.

Use these rule IDs when reporting architectural drift.

### AR-000 — Architecture rules are authoritative

The rules in this section must constrain specifications, plans, tasks, source code, tests, project structure, and module registration.

A generated artifact that violates these rules is architecturally misaligned.

Do not override these rules to make an implementation look simpler or more complete.

### AR-001 — Bounded contexts must be explicit modules

Every bounded context or business authority implemented in BrewUp must be represented as an explicit module.

A module requires:

- a solution folder;
- a physical directory under `src/<ModuleName>/`;
- a standard set of projects.

Do not implement a new bounded context as:

- a folder inside another module;
- a namespace inside another module;
- a set of classes inside `BrewUp.Shared`;
- a set of classes inside `BrewUp.Rest`;
- a set of classes inside the Sagas module.

If a feature introduces `Payment` as a separate authority and implementation is in scope, the plan must create a `Payment` module.

### AR-002 — Every business module uses the standard project structure

Every BrewUp business module must follow this project structure:

```text
src/<ModuleName>/
├── BrewUp.<ModuleName>.SharedKernel/
├── BrewUp.<ModuleName>.Domain/
├── BrewUp.<ModuleName>.ReadModel/
├── BrewUp.<ModuleName>.Infrastructure/
├── BrewUp.<ModuleName>.Facade/
└── BrewUp.<ModuleName>.Tests/
```

The projects must be added to the solution.

The projects must use the exact naming format:

```text
BrewUp.<ModuleName>.SharedKernel
BrewUp.<ModuleName>.Domain
BrewUp.<ModuleName>.ReadModel
BrewUp.<ModuleName>.Infrastructure
BrewUp.<ModuleName>.Facade
BrewUp.<ModuleName>.Tests
```

Do not use alternative names such as:

```text
BrewUp.<ModuleName>.Api
BrewUp.<ModuleName>.Application
BrewUp.<ModuleName>.Core
BrewUp.<ModuleName>.Contracts
BrewUp.<ModuleName>.Queries
```

unless the existing repository has already established that exact convention for the target module.

### AR-003 — SharedKernel owns module contracts

Each module owns its own `SharedKernel` project.

Module-specific IDs, commands, domain events, enums, and public message contracts belong in that module's `SharedKernel`.

For a new module, prefer this structure:

```text
BrewUp.<ModuleName>.SharedKernel/
├── DomainIds/
├── CustomTypes/
├── Enums/
└── Messages/
    ├── Commands/
    └── Events/
```

For the Payment module:

```text
BrewUp.Payment.SharedKernel/
├── DomainIds/
│   └── PaymentAuthorizationId.cs
├── Messages/
│   ├── Commands/
│   │   └── AuthorizePayment.cs
│   └── Events/
│       ├── PaymentAuthorized.cs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PacktPublishing/AI-Augmented-Domain-Driven-Design](https://github.com/PacktPublishing/AI-Augmented-Domain-Driven-Design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
