---
trigger: always_on
description: **Project:** DDD Meets GenAI — Reference Implementations
---

# AGENTS.md — Reference Implementation Conventions

**Project:** DDD Meets GenAI — Reference Implementations
**Authors:** Marden Neubert & Joseph Yoder
**Date:** March 2026

This file defines the architectural conventions and technology decisions for
generating reference implementations from DMML domain models. It is designed
to be read by any AI coding agent (Claude Code, Cursor, GitHub Copilot, Codex,
or similar) as the authoritative source of project conventions.

**The key design principle:** The DMML is the specification. This file defines
how to turn that specification into running code. Changing the decisions in
this file (language, framework, persistence, architecture) should produce a
different implementation from the _same_ DMML — without touching the domain
model. That separation is the whole point.

---

## 1. Technology Stack

| Concern | Choice | Rationale |
|---|---|---|
| Language | **TypeScript 5.x** | Accessibility for a broad audience. Strong typing for DDD patterns. |
| Runtime | **Node.js 22+** | LTS, native ESM, built-in test runner available as fallback. |
| HTTP Framework | **Fastify 5.x** | Lightweight, schema-first, fast. Less opinionated than NestJS. |
| Validation | **Zod 3.x** | Runtime schema validation with TypeScript inference. Maps cleanly to DMML attributes and invariants. |
| Testing | **Vitest 3.x** | Fast, ESM-native, Jest-compatible API. Excellent DX. |
| Package Manager | **pnpm** | Strict, fast, workspace support for monorepo. |
| Build | **tsup** or **tsc** | Simple bundling. No Webpack complexity. |
| Linting | **Biome** | Fast, opinionated, replaces ESLint + Prettier. |

### Why TypeScript Over Kotlin

Kotlin maps more naturally to DDD patterns (`data class`, `sealed class`,
`require()`). We chose TypeScript because: (1) broader audience accessibility
at XP 2026, (2) demonstrates that the DMML-to-code pipeline is language-
agnostic — the domain model doesn't care about the target language, and
(3) TypeScript's structural typing is "good enough" with disciplined use of
branded types and readonly patterns.

**This is a pluggable decision.** The same DMML + a Kotlin version of this
AGENTS.md file would produce a Kotlin/Spring Boot implementation. The domain
modeling work carries over completely.

---

## 2. Architecture: Hexagonal (Ports & Adapters)

Every bounded context follows hexagonal architecture as described by Vernon
(IDDD Chapter 4). The domain layer is the center with zero external
dependencies. Everything else depends inward.

```
┌─────────────────────────────────────────────────┐
│                  API Layer (Inbound Adapters)    │
│            Fastify routes → Application Services │
├─────────────────────────────────────────────────┤
│               Application Layer                  │
│       Use-case orchestration, event dispatch      │
├─────────────────────────────────────────────────┤
│                 Domain Layer                      │
│  Aggregates, Entities, Value Objects, Events,    │
│  Commands, Invariants, Repository Interfaces     │
├─────────────────────────────────────────────────┤
│           Infrastructure (Outbound Adapters)     │
│     In-memory repos, event publisher, ACLs       │
└─────────────────────────────────────────────────┘
```

### Dependency Rules

1. **Domain layer imports nothing** outside itself. No framework types, no
   infrastructure, no HTTP concepts. Pure TypeScript with Zod schemas.
2. **Application layer imports domain only.** It orchestrates aggregates,
   repositories (via interfaces), and event dispatch.
3. **Infrastructure implements domain interfaces.** Repository implementations,
   event publisher adapters, ACL translators.
4. **API layer calls application services.** Thin Fastify route handlers that
   translate HTTP requests into commands and return responses.

### Package Structure Per Bounded Context

Each bounded context is a self-contained package in the pnpm workspace:

```
packages/
  bc-order-management/
    src/
      domain/
        model/              # Aggregates, entities, value objects
          Order.ts          # Aggregate root
          OrderItem.ts      # Child entity
          Money.ts          # Value object
          OrderStatus.ts    # Lifecycle enum/union
        event/              # Domain events
          OrderPlaced.ts
          PaymentCompleted.ts
        command/            # Command types
          PlaceOrder.ts
          SelectPizzaType.ts
        repository/         # Repository interfaces (ports)
          OrderRepository.ts
        service/            # Domain services (if any)
      application/
        PlaceOrderService.ts    # Application service
        policy/                 # Event-driven policies
          PlaceOrderOnPayment.ts
      infrastructure/
        persistence/
          InMemoryOrderRepository.ts
        event/
          InProcessEventPublisher.ts
        acl/                    # Anti-corruption layers
          PaymentGatewayACL.ts
      api/
        routes/
          orderRoutes.ts    # Fastify route definitions
        schemas/            # Zod schemas for API validation
          orderSchemas.ts
    test/
      domain/               # Unit tests for domain logic
      application/          # Integration tests for use cases

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mardenneubert/ddd-meets-genai](https://github.com/mardenneubert/ddd-meets-genai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
