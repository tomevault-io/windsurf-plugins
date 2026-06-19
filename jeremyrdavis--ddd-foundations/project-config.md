---
trigger: always_on
description: >
---


# DDD Foundations Skill

The layering map and term definitions for any DDD-Quarkus bounded context. Every other DDD skill in this set
references the four layers and the canonical names below.

**Foundational principle.** Domain depends on nothing in your code — only the JDK and your own domain types.
Infrastructure and interfaces depend inward. The word "service" without a qualifier is meaningless: say
*application service* (use-case orchestration, owns the transaction, returns DTOs) or *domain service*
(stateless domain logic that doesn't naturally belong to one aggregate), and put each in the layer named after
it. The compiler enforces these layers via package boundaries — violations show up as imports, not as code
review findings.

**Red Flags — STOP if you find yourself thinking:**

- "Repository = persistence = infrastructure, so the interface goes there too."
- "It feels tidy to put the interface next to its implementation."
- "It's just a service — `service/` is fine for both kinds."
- "We're a small team / it's a small BC, let's keep it flat in one package and split later."
- About to write `import jakarta.persistence.*` or `import jakarta.ws.rs.*` in a class under `<bc>.domain`.
- About to write `import com.example.<bc>.infrastructure.*` from a class under `<bc>.domain` or `<bc>.application`.
- About to name a new class `FooService` without `Application` or `Domain` in the name.

If any of these surface, re-read Core Rules and Excuse / Reality before typing.

---

## When to Use

- Designing the package layout for a new bounded context.
- Placing a class (entity, value object, repository, domain service, application service, JAX-RS resource,
  ExceptionMapper).
- Naming a "service" class — choosing the layer-qualified suffix.
- Reviewing a diff that imports `jakarta.persistence.*` or `jakarta.ws.rs.*` in a `<bc>.domain` class.
- Reviewing a diff that puts a Repository interface in `<bc>.infrastructure`.
- Reviewing a diff where a JAX-RS resource injects a repository directly (skipping the application service).

**Out of scope**: the Quarkus mechanics inside any one layer (use the corresponding `quarkus-*` skill);
aggregate-internal design — invariants, factories, identity strategy (use `ddd-aggregates`); domain-event payload
and dispatch rules (use `ddd-domain-events`); choosing between active-record and repository pattern at the
persistence layer (architectural decision; the persistence skill takes the *Quarkus mechanics* of either path).

---

## Core Rules

1. **Four layers per bounded context**: `domain`, `application`, `infrastructure`, `interfaces` (often with a
   `interfaces.rest` sub-package). One bounded context = one top-level package; one set of layers under it.
2. **Domain depends on nothing in your code.** Only JDK types and types under `<bc>.domain` itself.
3. **Application depends on domain.** Application services orchestrate use cases by calling domain types and
   domain interfaces.
4. **Infrastructure depends on domain.** Infrastructure provides *implementations* of domain interfaces
   (repositories, external clients).
5. **Interfaces depend on application.** REST resources / gRPC handlers / CLI commands call application services
   and pass DTOs back and forth.
6. **Never the reverse.** Domain must not import application, infrastructure, or interfaces. Application must not
   import infrastructure or interfaces.
7. **Repository interfaces live in `<bc>.domain`**, alongside the aggregate they serve. The interface mentions
   only domain types — no `jakarta.persistence`, no Panache, no Hibernate.
8. **Repository implementations live in `<bc>.infrastructure`.** They implement the domain interface and bring in
   framework concerns. CDI wires them at runtime.
9. **Application service** = use-case orchestrator. Owns the transaction boundary (`@Transactional`). Returns
   DTOs, never entities. Lives in `<bc>.application`. Class name ends in `ApplicationService` (e.g.
   `AuthorizePaymentApplicationService`).
10. **Domain service** = stateless domain logic that doesn't naturally belong to one aggregate. No I/O, no
    framework calls, takes domain types and returns domain types. Lives in `<bc>.domain`. Class name ends in
    `DomainService` (e.g. `FraudDetectionDomainService`).
11. **Never name a class just `FooService`.** The unqualified name is the layer-confusion smell that lets
    transaction logic and domain rules drift between files in a shared `service/` folder.
12. **JAX-RS resources live in `<bc>.interfaces.rest`** and depend only on `<bc>.application`. They never
    `@Inject` a repository or an entity directly.
13. **DTOs (commands and read models) live in `<bc>.application`.** They cross the application↔interfaces
    boundary. The domain stays in domain types.

---

## Canonical Example

The package layout for a `payments` bounded context. Use case: *authorize a payment* — load the `Payment`
aggregate, run a domain-service fraud check, advance the aggregate's state machine, persist via a Panache
implementation, return a `PaymentDTO`.

```
com.example.payments
├── domain
│   ├── PaymentMethod              (value object — Java record)
│   ├── Payment                    (aggregate root)
│   ├── PaymentRepository          (interface — uses only domain types)
│   ├── FraudDetectionDomainService

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeremyrdavis/ddd-foundations](https://github.com/jeremyrdavis/ddd-foundations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
