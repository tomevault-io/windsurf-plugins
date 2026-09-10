---
trigger: always_on
description: handles 10k users
---

# CLAUDE.md

## Project

CommerceCore is a backend-only e-commerce system for studying correctness under concurrency and failure.

The focus is not storefront development.

The focus is:

```text
inventory
transactions
checkout
payments
idempotency
events
failure
recovery
```

Long-term progression:

```text
Catalog + Inventory
        ↓
Cart
        ↓
Inventory Reservations
        ↓
Orders + Checkout
        ↓
Idempotency
        ↓
Payments
        ↓
Transactional Outbox
        ↓
Event Delivery
        ↓
Saga / Compensation
        ↓
Reconciliation
        ↓
Failure Experiments
```

## Stack

Primary stack:

```text
Java 21
Spring Boot
Gradle
PostgreSQL
JUnit
Testcontainers
```

Local dependencies may use:

```text
Docker
Docker Compose
```

Kafka and Redis may be introduced later only when an implemented requirement justifies them.

Kubernetes is not part of the core project.

## Architecture

Begin as a modular monolith.

One application.

One PostgreSQL database.

Keep domain boundaries clear.

Do not split modules into microservices merely because they might eventually communicate over a network.

Distributed boundaries should be introduced deliberately so their costs can be studied.

## Primary priorities

Optimize for:

1. business correctness
2. transactional correctness
3. concurrency correctness
4. idempotency
5. explicit state transitions
6. reproducible failures
7. integration tests using real dependencies
8. understandable code

Do not optimize for architecture complexity.

## Git policy

Do not commit unless explicitly instructed.

Do not push unless explicitly instructed.

Never:

```text
force push
rewrite history
rebase without approval
stage unrelated files
```

Before a requested commit report:

```text
files changed
diff summary
tests
proposed commit message
```

## No AI attribution

Never add references to:

```text
Claude
Anthropic
ChatGPT
OpenAI
Copilot
AI-generated
generated-by
assisted-by
```

Never add an AI system as an author, contributor, or co-author.

Never add AI `Co-Authored-By` trailers.

## Avoid AI slop

Do not automatically create:

```text
Controller
Service
Repository
DTO
Mapper
Interface
Impl
Factory
Manager
```

for every domain concept.

Every layer must have a reason.

Avoid:

```text
BaseCrudService
GenericRepository
CommerceManager
BusinessEngine
DomainManager
```

and other abstractions that hide the actual commerce behavior.

Prefer explicit domain concepts.

## Spring

Use Spring as infrastructure.

Do not allow Spring conventions to obscure business invariants.

The question:

> Why is this operation correct?

should be answerable from the actual domain/database behavior, not simply:

> because Spring handles it.

## PostgreSQL

PostgreSQL is the authoritative persistent store initially.

Use real PostgreSQL for persistence/concurrency integration tests.

Do not replace it with H2 when PostgreSQL semantics matter.

Use Flyway for schema migrations.

Do not depend on Hibernate auto-creating the production schema.

## Database invariants

Protect important invariants at the database layer when appropriate.

Examples:

```text
unique SKU
inventory >= 0
unique idempotency key
unique external payment event
```

Application checks alone may race.

Database constraints are part of the design.

## Concurrency

Do not solve cross-request correctness with only:

```java
synchronized
```

or another process-local lock.

Correctness should survive multiple application instances whenever the database can enforce it naturally.

Use explicit PostgreSQL concurrency mechanisms.

Examples may include:

```text
conditional UPDATE
row locking
unique constraint
optimistic versioning
```

Choose based on the actual invariant.

Do not introduce distributed locks automatically.

## Transactions

A transaction must protect a specific invariant.

Do not add `@Transactional` everywhere by habit.

For each important transaction be able to explain:

```text
what must change atomically?
what failure would occur without the transaction?
```

## Money

Never use:

```text
double
float
```

for money.

Use a deliberate representation such as:

```text
BigDecimal with explicit scale
```

or integer minor units.

Document currency assumptions.

Do not build multi-currency support before needed.

## Inventory

Inventory must never become negative.

Cart quantity is not inventory reservation.

A cart does not guarantee stock.

Later reservation state must be explicit.

Do not conflate:

```text
cart
available inventory
reserved inventory
sold inventory
```

## Idempotency

Retries are normal backend behavior.

When checkout idempotency exists, correctness must persist across application restart.

Do not implement idempotency using only process-local memory.

Distinguish:

```text
same request retried
```

from:

```text
new logical operation
```

## Payments

Treat payment providers as external systems that may:

```text
timeout
retry
duplicate callbacks
respond late
return ambiguous outcomes
```

Never assume a timeout means payment failed.

Do not blindly retry potentially successful payment operations.

Later use reconciliation when provider outcome is ambiguous.

## Webhooks

Assume webhook delivery is at least once.

Handlers must tolerate duplicates.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nhatminh06/commercecore](https://github.com/nhatminh06/commercecore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
