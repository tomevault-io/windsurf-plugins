---
trigger: always_on
description: Contract-first architecture — design driven by specifications, C4 model, DDD patterns
---


# Contract-First Architecture

## Decision Framework

For every architectural decision, document as an ADR referencing the specs that drove it:

1. **Context**: What spec requirement or contract constraint drives this decision?
2. **Options**: What are the viable approaches? (minimum 2)
3. **Trade-offs**: Pros and cons of each option against the spec requirements.
4. **Decision**: Which option and why — with spec references.
5. **Consequences**: What contracts are affected? What new specs are needed?

## Architecture Patterns

Choose based on the contracts defined in the Specification phase:

| Pattern | When to Use | Spec Signal |
|---------|-------------|-------------|
| **Monolith** | Small team, single product, few API contracts | <10 endpoints, single data store |
| **Modular Monolith** | Growing product, clear contract boundaries | Multiple bounded contexts in specs |
| **Microservices** | Independent contracts, different scaling needs | Separate API specs per domain |
| **Serverless** | Event-driven contracts, async workflows | AsyncAPI specs, event schemas |
| **Jamstack** | Content-heavy, static + dynamic hybrid | Few API contracts, CDN-friendly |
| **Event-Driven** | Async workflows, decoupled contracts | Event schemas, pub/sub contracts |

## C4 Architecture Model

Document architecture at four levels of abstraction, driven by specs:

### Level 1: System Context Diagram

Shows the system and its external actors/dependencies. Derived from integration contracts.

```
┌─────────────────────────────────────────────────────┐
│                   SYSTEM CONTEXT                     │
│                                                      │
│  [User] ──→ [Your System] ──→ [Payment Provider]    │
│                  │                                    │
│                  ├──→ [Email Service]                 │
│                  └──→ [Auth Provider]                 │
│                                                      │
│  Sources: specs/contracts/*.pact.json                │
│           specs/api/openapi.yaml (security schemes)  │
└─────────────────────────────────────────────────────┘
```

Rules:
- Every external dependency must have an integration contract.
- Every actor must be referenced in behavior specs.

### Level 2: Container Diagram

Shows the major containers (apps, databases, queues). Derived from API specs and data contracts.

```
┌────────────────────────────────────────────────────────────┐
│                    CONTAINER DIAGRAM                        │
│                                                             │
│  [Web App (Next.js)] ──API──→ [API Server (Node.js)]       │
│                                     │                       │
│                              ┌──────┼──────┐                │
│                              ↓      ↓      ↓                │
│                          [PostgreSQL] [Redis] [S3]          │
│                                                             │
│  [Mobile App] ──API──→ [API Server]                         │
│                                                             │
│  Sources: specs/api/openapi.yaml (API boundaries)           │
│           specs/schemas/ (data store decisions)              │
│           specs/decisions/ (ADRs)                            │
└────────────────────────────────────────────────────────────┘
```

### Level 3: Component Diagram

Shows modules within a container. Derived from feature specs and bounded contexts.

```
┌──────────────────────────────────────────────────────────┐
│              API SERVER — COMPONENT DIAGRAM               │
│                                                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐           │
│  │   Auth   │  │  Users   │  │   Orders     │           │
│  │  Module  │  │  Module  │  │   Module     │           │
│  └────┬─────┘  └────┬─────┘  └──────┬───────┘           │
│       │              │               │                    │
│  ┌────┴──────────────┴───────────────┴────┐              │
│  │         Shared Infrastructure           │              │
│  │  (middleware, error handler, logger)     │              │
│  └─────────────────────────────────────────┘              │
│                                                           │
│  Sources: specs/features/ (one module per feature area)   │
│           specs/schemas/ (shared schemas = shared infra)  │
└──────────────────────────────────────────────────────────┘
```

### Level 4: Code Diagram

Shows classes/functions within a component. Derived from data contracts and behavior specs. Use sparingly — code should be self-documenting at this level.

## Domain-Driven Design Patterns (Spec-Driven)

For complex domains, use DDD patterns mapped to specs:

### Bounded Contexts

Each bounded context maps to a separate set of specs:

```
specs/
  contexts/
    identity/               # Auth & user management context
      api/openapi.yaml
      schemas/user.schema.json
      features/auth.feature
    ordering/               # Order management context
      api/openapi.yaml
      schemas/order.schema.json
      features/checkout.feature
    shipping/               # Shipping context
      api/openapi.yaml
      schemas/shipment.schema.json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaetanOff/WAF-GaetanDev](https://github.com/GaetanOff/WAF-GaetanDev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
