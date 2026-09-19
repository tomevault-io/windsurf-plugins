---
trigger: always_on
description: > **IMPORTANT:** Use retrieval-led reasoning. Read the relevant source and documents before you decide. Do not infer project behavior from general knowledge.
---

# Agentic Identity Broker — Agent Context

> **IMPORTANT:** Use retrieval-led reasoning. Read the relevant source and documents before you decide. Do not infer project behavior from general knowledge.

## How to Use This File

Use this routing document to find repository areas. Before you change an area, read its nearest `AGENTS.md`. Use the index for detailed documents.

## Docs Index (Retrieval Targets)

For compact scanning, use pipe-delimited entries. Before you implement in a domain, read the relevant file.

### Architecture & Decisions

`ARCHITECTURE.md` | Source of truth for system design
`adrs/NNN-*.md` | Binding ADRs — read before implementation (see the ADR index that follows)
`.specify/memory/constitution.md` | Binding constitution — 13 principles governing all work

### Section-Specific Agent Context

`internal/AGENTS.md` | Backend hexagonal architecture, builder pattern, testing conventions
`internal/domain/AGENTS.md` | Domain layer rules, zero-infra imports, data models, error types
`internal/domain/id/AGENTS.md` | Strongly typed entity IDs, code generation, type catalogue
`internal/ports/AGENTS.md` | Port interface catalogue, ISP rules, error conventions
`internal/adapters/AGENTS.md` | Adapter map, cross-adapter ban, storage/encryption/HTTP details
`internal/extproc/AGENTS.md` | Standalone ExtProc token exchange service, config, architecture
`web/AGENTS.md` | React SPA, design system, API client, testing
`infra/AGENTS.md` | AWS CDK encryption stack, IRSA, environment parameterization
`tests/AGENTS.md` | E2E + integration test suite overview
`tests/e2e/AGENTS.md` | Ginkgo E2E rules, fixtures, dual-server pattern, anti-patterns
`tests/e2e/frontend/AGENTS.md` | Playwright browser tests, page objects
`tests/integration/AGENTS.md` | Self-contained and infrastructure-backed integration test guidance

### Runtime Entry Points

`cmd/agentic-identity-broker/` | Broker CLI, configuration, and dual-server startup
`cmd/extproc-token-exchange/` | ExtProc CLI, telemetry bridge, and gRPC startup

### API Contracts

`api/enduser/openapi.yaml` | End-user API: consent, OAuth2, sessions, and approvals
`api/admin/openapi.yaml` | Admin API: agents, services, resources, and permission sets

### Specs (Feature Requirements)

`specs/NNN-<name>/spec.md` | Feature requirements + acceptance scenarios
`specs/NNN-<name>/plan.md` | Implementation approach
`specs/NNN-<name>/tasks.md` | Trackable task list

### Encryption Deep Reference

`.claude/skills/aws-crypto-go/SKILL.md` | AWS Encryption SDK Go — patterns, rules
`.claude/skills/aws-crypto-go/reference.md` | API reference
`.claude/skills/aws-crypto-go/examples.md` | Code examples

### Design System (Frontend)

`web/src/design-system/docs/INDEX.md` | Complete design system documentation index
`web/src/design-system/docs/COMMON_MISTAKES.md` | Read before you work on a styled component
`web/src/design-system/docs/DECISION_TREES.md` | Which component to use when

### Operations & Guides

`docs/ENCRYPTION_INTEGRATION_GUIDE.md` | Encryption integration guide
`docs/STORAGE_EXTENSION_GUIDE.md` | Adding new storage entities
`docs/STORAGE_TROUBLESHOOTING.md` | Storage debugging
`docs/configuration.md` | Configuration reference
`docs/deployment/` | Deployment guides (Kubernetes, IRSA)
`docs/operations/` | Operations runbooks
`examples/config/` | Example configuration files
`charts/agentic-identity-broker/` | Helm deployment contract
`migrations/` | PostgreSQL schema migrations
`mocks/` | Standalone test services and OAuth2 clients

## Constitution (Binding) — 13 Principles

Treat `.specify/memory/constitution.md` as **BINDING**. Apply these principles:

1. **Security-First** — Fail closed. Do not use bypasses. Never treat signature validation as optional.
2. **ADRs are Binding** — Treat ADRs as binding. Use `ARCHITECTURE.md` as the source of truth. A deviation requires a superseding ADR.
3. **Library-First Security** — Do not use custom crypto. Use Go `crypto/*`, `golang.org/x/crypto`, or AWS Encryption SDK.
4. **OpenAPI Transparency** — Define APIs in `api/{enduser,admin}/openapi.yaml` before implementation.
5. **Domain-Driven Design** — Enforce ubiquitous language. Add new concepts to the `ARCHITECTURE.md` glossary.
6. **Hexagonal Architecture** — Arrange dependencies as domain → ports (interfaces) → adapters. Never reverse this order. Prevent these violations:
   - **Port bypass**: Route handlers through domain services to ports. Do not route a handler to a port directly.
   - **Anemic domain**: Make services enforce invariants. Do not only proxy port calls.
   - **Domain logic leakage**: Put conditional logic beyond input parsing in domain services, not handlers.
   - **Domain packaging**: Make each new `domain/X/` a genuinely independent bounded context.
7. **Configuration-Driven** — Get all config through `internal/ports/config.go`. Do not load config ad hoc.
8. **TDD** — Use red-green-refactor. Write tests first. Make each test fail before implementation.
9. **Persistence Consistency** — Define ISP repos in `internal/ports/storage.go`. Use sqlx for PostgreSQL. Provide both in-memory and postgres adapters.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zalando-incubator/agentic-identity-broker](https://github.com/zalando-incubator/agentic-identity-broker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
