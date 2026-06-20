---
trigger: always_on
description: >
---


# DDD Value Objects Skill

How to model value objects in a Quarkus DDD project. A value object has no identity, is immutable, equals by
value, and encodes its invariants in its construction. Records are the canonical shape.

**Foundational principle.** A value object encodes its invariants in its constructor. If validation lives
anywhere except the type's constructor, the type is just a container for primitives wearing a class label.
Records are the canonical shape: immutable, value-based equality, validation in the canonical constructor,
replacement (not mutation) via new construction. **Value objects are plain Java records — no
`jakarta.persistence.*` imports, no `@Embeddable`, no `@Id`.** Domain depends on nothing in your code (per
`ddd-foundations`); the persistence layer maps value-object records to columns or to its own `@Embeddable`
mirror class in `<bc>.infrastructure` (covered by `ddd-repositories`). The named failure mode this skill
prevents is **primitive obsession** — modeling a domain concept (money, email, postal address, order ID) as
raw `BigDecimal` / `String` / `Long` fields scattered across the aggregate.

**Red Flags — STOP if you find yourself thinking:**

- About to declare `BigDecimal amount` and `String currency` (or `Currency currency`) as **separate** fields on
  an entity — that's a `Money` value object trying to be born.
- About to use raw `String email`, `String sku`, `String phoneNumber`, or `String addressLine1` on a domain
  class — that's `EmailAddress`, `Sku`, `PhoneNumber`, `AddressLine1` (or just `Address`) trying to be born.
- About to write a JavaBean class with setters and call it a "value object."
- About to hand-write `@Override boolean equals(Object)`, `int hashCode()`, `String toString()` — records do
  this for free.
- About to add a `*Validator` companion class to validate a value-shaped type.
- About to add a method that mutates the value's fields (`this.amount = this.amount.add(...)`).
- About to put `@Data` (Lombok) on a value-object class.
- About to add `@Id` to a value-object record — that gives it identity, which makes it an entity.
- About to write `import jakarta.persistence.*` (or any JPA annotation) on a value-object record in `<bc>.domain`.
- "I'll mark this record `@Embeddable` so the persistence layer can pick it up directly — saves a mapper."
- "JPA needs a no-arg constructor and setters, so records are awkward."
- "BigDecimal and String are simpler — a wrapper is over-engineering."

If any of these surface, re-read Core Rules and Excuse / Reality before typing.

---

## When to Use

- Designing a new value object: `Money`, `EmailAddress`, `Address`, `PhoneNumber`, `IpAddress`, `Coordinates`.
- Designing a typed ID wrapper used as a value at API surfaces: `OrderId(UUID)`, `CustomerId(UUID)`,
  `Sku(String)`.
- Adding behavior to a value object (`Money.plus`, `Money.times`, `EmailAddress.domain`).
- Reviewing a diff with raw primitives where a value object would cohere (`BigDecimal amount` + `String currency`),
  hand-written `equals`/`hashCode` on a value-shaped class, a `*Validator` companion for a `Money`-shaped type,
  mutating methods on a class that should be immutable, or **`jakarta.persistence.*` imports on a value-object
  record in `<bc>.domain`** (the value object is a plain record; mapping is the persistence layer's job).

**Out of scope**: layering and package placement (use `ddd-foundations`); aggregate-root design (use
`ddd-aggregates` — it covers identity, factory methods, state machines); the persistence-layer mapping that
turns a value-object record into columns, or into a sibling `@Embeddable` mirror class in `<bc>.infrastructure`
(use `ddd-repositories`); domain-event payload conventions (use `ddd-domain-events` — events are records but
their *naming and dispatch* live there); the Quarkus mechanics of `@Embeddable` records on Hibernate (covered
by `quarkus-persistence`); domain services (use `ddd-services`).

---

## Core Rules

1. **A value object is a Java record.** That gives you immutability, value-based `equals`/`hashCode`/`toString`,
   and a compact declaration without boilerplate. Don't write a class for a value object unless you have a
   measured reason the record can't satisfy.
2. **Validate in the canonical (compact) constructor.** `public Money { Objects.requireNonNull(currency); if
   (amount.signum() < 0) throw ...; }` — the validation runs every time the record is constructed, including
   when Hibernate hydrates it from a row. There is no "valid path" and "invalid path"; there is one path.
3. **Normalize in the canonical constructor by reassigning the parameter.** Records allow `value = value.trim()
   .toLowerCase()` inside the canonical constructor — the canonical-constructor reassignment is the idiom for
   trimming, lowercasing, scaling decimals, etc.
4. **Equality is by value.** Records produce equals/hashCode that compare every component. Don't override them.
   If two value objects with the same components must compare *unequal*, you have an entity (it has identity);
   use the aggregate-root pattern (covered by `ddd-aggregates`) instead.
5. **Value objects are immutable.** No setters. No methods that change `this`. Operations that "modify" the
   value return a *new* value: `Money.plus(other)` returns a new `Money`, never mutates the receiver. Records

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeremyrdavis/ddd-value-objects](https://github.com/jeremyrdavis/ddd-value-objects) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
