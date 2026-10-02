---
trigger: always_on
description: generateStructured<TInput, TOutput>(
---

# CLAUDE.md

This file is the operating guide for Claude Code and any coding agent working in this repository.

Read this file before changing code.

---

## 1. Project mission

Build **ARF-OS**, a multi-agent research operating system for discovering, implementing, backtesting, validating, and forward-testing systematic trading strategies.

The system is a research platform. It is not an autonomous fund, broker, or live-trading engine.

The core product promise is:

> Every strategy result is reproducible, versioned, independently validated, and traceable from idea to decision.

Pine Script v6 is the canonical strategy language. A Pine-compatible local runner may be used for scale, but TradingView is the final acceptance and parity environment for Pine behaviour.

---

## 2. Read these documents

Before implementing a feature, read:

1. `AI_RESEARCH_HEDGE_FUND_SPEC.md`
2. `LEADER_AGENT_SYSTEM_PROMPT.md`
3. Relevant schemas in `schemas/`
4. Relevant ADRs in `docs/adr/`
5. Existing tests near the code you will change

When implementation and specification conflict, do not silently choose one. Open or update an ADR and make the conflict visible.

---

## 3. Non-negotiable architecture rules

### 3.1 Immutable research artefacts

Never mutate a tested strategy version.

Any change to any of the following creates a new `strategy_versions` row:

- Pine source
- Strategy definition
- Parameters
- Symbol
- Timeframe
- Session or timezone
- Cost model
- Position sizing
- Leverage or margin
- Execution settings
- Dataset
- Runner
- Segment assignment

Backtest and forward-test records point to the exact immutable version.

### 3.2 API owns state transitions

Workers do not directly change strategy lifecycle state.

Workers:

1. execute a job,
2. store output artefacts,
3. emit a domain event or result,
4. let the orchestrator/API apply transition policy.

### 3.3 Structured contracts are canonical

All agent outputs, handoffs, domain events, and external signal payloads must be validated with Zod.

Do not accept critical free-form model output and then infer fields with regular expressions.

Prose summaries may accompany structured data, but structured data is canonical.

### 3.4 Separation of duties

Enforce role separation in code:

- Creator cannot be sole validator.
- Validator cannot edit source.
- Strategy Judge cannot grant live approval.
- Forward operator cannot change active strategy parameters.
- Human override must be explicit and audited.

### 3.5 Protected data

Final holdout and forward data require explicit access checks.

Never return protected results through a general strategy endpoint without verifying role and stage.

Every protected-data read writes an audit event.

### 3.6 Idempotency

Every side-effecting command and background job must be idempotent.

Use:

- request idempotency keys,
- deterministic job keys,
- unique database constraints,
- event IDs,
- deployment signal IDs.

Retries must not create duplicate strategy versions, backtests, paper orders, fills, or decisions.

### 3.7 No hidden business logic in prompts

Prompts may guide agent behaviour, but promotion gates, access control, lifecycle transitions, budgets, and hard-fail rules live in deterministic application code.

A model recommendation is evidence, not authority.

### 3.8 No browser automation as a core dependency

The MVP supports human-assisted TradingView verification and CSV ingestion.

Do not build the platform around fragile UI selectors or unattended browser automation unless an approved ADR explicitly documents terms, reliability, and security implications.

### 3.9 No live trading in the initial product

Do not add exchange credentials, order-routing code, or live execution side effects without a separately approved project specification.

Paper execution must be clearly named and isolated.

---

## 4. Technology choices

Use the current stable versions pinned in the repository lockfile and runtime configuration.

### Applications

- `apps/web`: Next.js App Router
- `apps/api`: Fastify
- `apps/worker-research`: model and research jobs
- `apps/worker-backtest`: runner and report ingestion jobs
- `apps/worker-analytics`: metrics and robustness jobs
- `apps/worker-forward`: TradingView webhook and paper-test jobs

### Packages

- TypeScript
- pnpm workspaces
- Turborepo
- PostgreSQL
- Drizzle ORM
- Redis
- BullMQ
- Zod
- Clerk
- OpenTelemetry
- Vitest
- Playwright
- S3-compatible object storage

Avoid introducing a large framework when a small internal abstraction is sufficient.

The agent orchestration state machine belongs to ARF-OS. Do not hide it inside a third-party agent framework.

---

## 5. Expected repository layout

```text
/
├── apps/
│   ├── web/
│   ├── api/
│   ├── worker-research/
│   ├── worker-backtest/
│   ├── worker-analytics/
│   └── worker-forward/
├── packages/
│   ├── contracts/
│   ├── db/
│   ├── agent-runtime/
│   ├── workflow/
│   ├── metrics/
│   ├── pine/
│   ├── backtest-sdk/
│   ├── event-bus/
│   ├── auth/
│   ├── observability/
│   └── ui/
├── pine/
│   ├── boilerplate/
│   ├── libraries/
│   ├── fixtures/
│   └── generated/
├── schemas/
├── docs/
│   └── adr/
├── infra/
├── AI_RESEARCH_HEDGE_FUND_SPEC.md
├── LEADER_AGENT_SYSTEM_PROMPT.md
└── CLAUDE.md
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daviddme/AIHedgeFund](https://github.com/daviddme/AIHedgeFund) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
