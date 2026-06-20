---
trigger: always_on
description: >
---


# DDD Services Skill

Two kinds of services, two layers, two jobs. **Application services orchestrate** (load, mutate, persist,
dispatch — own the transaction). **Domain services compute** (pure logic over domain inputs, no I/O, no
framework). The class-name suffix and the package must agree.

**Foundational principle.** Aggregates guard invariants. Domain services compute across inputs. Application
services orchestrate use cases. Each one belongs in a different layer with different dependencies allowed. A
class named `*DomainService` that injects `@Transactional` and a repository is an application service wearing
the wrong hat — the suffix is a label, not a license. A class named just `*Service` (no qualifier) is the
layer-confusion smell that lets transaction logic creep into pure rules and pure rules creep into orchestration.

**Red Flags — STOP if you find yourself thinking:**

- About to name a class `OrderService` (no `Application` or `Domain` qualifier) — pick one and put it in the
  matching package.
- About to put `@Transactional` on a class named `*DomainService`.
- About to `@Inject` a repository, an `Event<>`, an HTTP client, a Kafka producer, or any other I/O thing into
  a `*DomainService`.
- About to put a `*DomainService` in `<bc>.application` (or a `*ApplicationService` in `<bc>.domain`).
- About to inline 50+ lines of business rules as a `private` method inside an application service.
- About to put a multi-line policy calculation inside an aggregate root method that should just enforce a state
  transition (e.g. an 80-line refund-amount calculation inside `Order.refund(...)`).
- About to make a domain service `static` to "avoid the CDI ceremony" — domain services are CDI-managed
  `@ApplicationScoped` classes; the discipline is *what's injected into them* (nothing framework-bound), not
  whether they're managed.

If any of these surface, re-read Core Rules and Excuse / Reality before typing.

---

## When to Use

- Designing a new service class for any use case.
- Naming a class that ends in "Service" — picking `*ApplicationService` vs `*DomainService` vs neither (and
  putting the logic on the aggregate instead).
- Reviewing a diff with a `*DomainService` that has `@Transactional` or an injected repository.
- Reviewing a diff where business rules are inlined as a private method in an application service (often
  signals a missing domain service).
- Reviewing a diff where an aggregate root method computes policy in addition to enforcing the state
  transition.

**Out of scope**: layering and package placement (use `ddd-foundations` — this skill assumes the
`<bc>.application` and `<bc>.domain` packages and dependency directions); aggregate-root invariants and state
machines (use `ddd-aggregates`); repository interface design (use `ddd-repositories`); domain-event payload and
dispatch mechanics (use `ddd-domain-events` — this skill mentions that the application service drains and
dispatches, but the *event shape and observer rules* live there); the Quarkus mechanics of `@Transactional`
placement and CDI scopes (use `quarkus-persistence`).

---

## Core Rules

1. **Two kinds of services, two suffixes, two layers.** `*ApplicationService` lives in `<bc>.application`.
   `*DomainService` lives in `<bc>.domain`. Never name a class just `*Service` — the unqualified name is a
   layer-confusion smell.
2. **Application service = use-case orchestration.** Loads aggregates via repository interfaces, calls domain
   methods on them, optionally calls a domain service for cross-aggregate computation, persists via the
   repository, drains and dispatches `pendingEvents`, maps the result to a DTO, and returns. Owns the
   `@Transactional` boundary (per `quarkus-persistence`).
3. **Application service may inject:** repository interfaces, other application services (sparingly), domain
   services, `Event<DomainEvent>`, `Clock` and other framework-supplied infrastructure. It is the correct
   place to mention `@Transactional`, `Event`, and any I/O bean.
4. **Domain service = pure logic that doesn't naturally belong to one aggregate.** Computes across multiple
   aggregates / values / a `Clock` / domain configuration. Returns a domain value (a value object, a verdict,
   a calculated `Money`). No state changes, no persistence, no I/O.
5. **Domain service may inject:** other domain services and CDI-supplied configuration values that are domain
   data (e.g. a `RefundPolicy` configured for the BC). It must not inject repositories, `Event<>`, HTTP/Kafka
   clients, `EntityManager`, `@Transactional` propagation. The dependency rule from `ddd-foundations` (domain
   depends on nothing) is the same rule expressed at this level.
6. **Default first: put logic on the aggregate.** Most domain logic should live inside the aggregate root, on
   the aggregate's domain methods, where it's adjacent to the state it operates on. Extract a domain service
   only when the logic *doesn't naturally belong to one aggregate* — typically because it spans two aggregates
   or two value objects, or because it needs an external input (a `Clock`, a configured policy) the aggregate
   doesn't carry.
7. **Aggregates guard invariants; domain services compute across inputs.** This is the heuristic for choosing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeremyrdavis/ddd-services](https://github.com/jeremyrdavis/ddd-services) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
