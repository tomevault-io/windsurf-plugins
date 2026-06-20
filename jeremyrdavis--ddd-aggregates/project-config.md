---
trigger: always_on
description: >
---


# DDD Aggregates Skill

How to design an aggregate root in a Quarkus DDD project. The aggregate is **a plain POJO** in `<bc>.domain`
that enforces its own invariants and is the boundary of one transaction. The persistence-layer JPA entity is a
*separate* class in `<bc>.infrastructure` (covered by `ddd-repositories`); the aggregate has no knowledge of it.

**Foundational principle.** Domain depends on nothing in your code (per `ddd-foundations`) — and that includes
`jakarta.persistence.*`. The aggregate is a plain POJO with no JPA annotations, no Panache extension, no
`EntityManager` reference. The aggregate enforces its own invariants; anything outside it that "validates"
before mutating is a sign that the invariant doesn't actually live there. The aggregate's public surface is the
set of legal operations; private state plus domain methods is how you guarantee that surface is exhaustive.
Public fields, public setters, external `*Validator` classes, and JPA annotations on the aggregate are
different shapes of the same anti-pattern: they let infrastructure or untrusted code shape the domain layer.

**Red Flags — STOP if you find yourself thinking:**

- About to write `import jakarta.persistence.*` (or any `jakarta.persistence.*` annotation) on a class in
  `<bc>.domain`.
- About to write `extends PanacheEntity` / `extends PanacheEntityBase` on an aggregate.
- About to declare a `public` mutable field on an aggregate ("Panache active-record idiom").
- About to add a `setStatus(...)` (or any other setter) on an aggregate root.
- About to add a separate `*Validator` class that "checks the entity is valid" before persistence.
- About to reference another aggregate root by object (`Customer customer` field) instead of by ID
  (`CustomerId customerId`).
- About to call `new Order(...)` from a service method instead of `Order.place(...)` (or another named factory).
- About to put `@Transactional` on a method *inside* the aggregate — the aggregate is a POJO and doesn't
  know about transactions.
- "Mapping the aggregate to a separate JPA entity duplicates the field list — DRY says one type."
- "Panache's idiom is public fields — why fight the framework?"
- "It's a small / CRUD-shaped service — the rich-aggregate ceremony is over-engineering."

If any of these surface, re-read Core Rules and Excuse / Reality before typing.

---

## When to Use

- Designing a new aggregate root or modifying an existing one.
- Picking the identity strategy: typed UUID wrapper, externally-assigned identifier, or — only when the ID has
  no domain meaning — a plain `Long`.
- Writing the static factory method that constructs a valid initial state.
- Writing the `rehydrate(...)` static method the persistence layer reconstructs from.
- Deciding whether an invariant lives in the aggregate or somewhere else.
- Choosing how to reference another aggregate root (always by ID).
- Adding a `long version` field for optimistic locking.
- Raising a domain event from inside an aggregate when state changes.
- Reviewing a diff with any `jakarta.persistence.*` import or Panache extension on a class in `<bc>.domain` —
  that's a category error this skill prevents.

**Out of scope**: layering and package placement (use `ddd-foundations`); the internal shape and equality of
value objects, including the typed ID wrappers `OrderId`, `CustomerId` (use `ddd-value-objects`); the
persistence-layer JPA entity that mirrors the aggregate, the repository interface, and the repository
implementation that translates between domain and JPA types (use `ddd-repositories`); domain-event payload
conventions and CDI dispatch (use `ddd-domain-events`); application-service orchestration and `@Transactional`
placement (use `ddd-services` and `quarkus-persistence`); the Quarkus mechanics of `@Embeddable` records and
Panache base classes — *those concerns belong to the persistence layer, not the aggregate* (covered in
`quarkus-persistence`).

---

## Core Rules

1. **The aggregate is a plain POJO.** No `jakarta.persistence.*` imports. No `@Entity`, `@Id`, `@Version`,
   `@Embedded`, `@ElementCollection`, `@Transient`. No `extends PanacheEntity` / `PanacheEntityBase`. No
   `EntityManager` reference. The aggregate lives in `<bc>.domain` and depends on the JDK plus its own domain
   types (per `ddd-foundations`). The persistence-layer JPA entity is a **separate class** in
   `<bc>.infrastructure` and the repository implementation maps between them (covered by `ddd-repositories`).
2. **The aggregate is the only entry point to its internal state.** No public fields, no setters. Mutation
   happens through named domain methods (`cancel()`, `markFulfilled()`, `addLine(...)`) that enforce the
   invariants of that operation.
3. **Private state, public domain methods.** Fields are `private`; expose what callers need via accessor methods
   (`status()`, `total()`) — not via getters that mirror every field. The aggregate decides what's visible.
4. **Construct via a static factory, not `new`.** A method like `Order.place(id, customerId, lines)` validates
   inputs and returns a fully-constructed, valid aggregate. The constructor is `private`; external callers
   never see it.
5. **The factory enforces creation invariants.** Empty line list? Mixed currencies? Missing customer ID? The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeremyrdavis/ddd-aggregates](https://github.com/jeremyrdavis/ddd-aggregates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
