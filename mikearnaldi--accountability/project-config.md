---
trigger: always_on
description: This document provides guidance for Claude Code when working on the Accountability project.
---

# Accountability - Claude Code Guide

This document provides guidance for Claude Code when working on the Accountability project.

## Project Overview

Accountability is a multi-company, multi-currency accounting application using:
- **Effect** - Functional TypeScript library for type-safe backend business logic (server only)
- **TanStack Start** - Full-stack React framework with SSR and file-based routing
- **openapi-fetch** - Typed fetch client generated from Effect HttpApi's OpenAPI spec

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      FRONTEND (web)                         │
│  React + TanStack Start + openapi-fetch + Tailwind          │
│  NO Effect code - loaders for SSR, useState for UI          │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ HTTP (openapi-fetch client)
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       BACKEND (api)                          │
│  Effect HttpApi + HttpApiBuilder                             │
│  Exports OpenAPI spec for client generation                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   PERSISTENCE + CORE                         │
│  @effect/sql + PostgreSQL │ Effect business logic            │
└─────────────────────────────────────────────────────────────┘
```

## 🚨 CRITICAL: BACKEND AND FRONTEND MUST STAY ALIGNED 🚨

**This is a HARD REQUIREMENT. When implementing features:**

1. **NEVER do frontend-only changes** for features that need backend work
2. **ALWAYS update both layers** - packages/web AND packages/core, packages/api, packages/persistence
3. **Frontend workarounds are NOT acceptable** - if the spec says "update API", update the API
4. **Run tests** - `pnpm test && pnpm typecheck` MUST pass before marking work complete
5. **Read specs/UI_ARCHITECTURE.md** - See "MANDATORY: BACKEND AND FRONTEND MUST STAY ALIGNED" section

**Data flow**: Frontend → API → Service → Repository → Database
**All layers must be consistent.**

## 🚫 NEVER RUN DOCKER

**NEVER run docker commands.** The database and infrastructure are managed externally. Tests use testcontainers which handle their own containers automatically. Do not run:
- `docker run`
- `docker compose`
- `docker-compose`
- Any docker-related commands

## Project Structure

```
accountability/
├── packages/
│   ├── core/           # Core accounting logic (Effect, 100% tested) ← BACKEND
│   ├── persistence/    # Database layer (@effect/sql + PostgreSQL) ← BACKEND
│   ├── api/            # Effect HttpApi server + OpenAPI export ← BACKEND
│   └── web/            # React UI (NO Effect - loaders + openapi-fetch client) ← FRONTEND
├── specs/              # All specs and context - use focus mode to select relevant files
└── repos/              # Reference repositories (git subtrees)
    ├── effect/         # Effect-TS source
    └── tanstack-router/# TanStack Router/Start source
```

## Specs Directory

All specifications and context documentation live in `specs/`. Use focus mode to select relevant files for your task.

### Consolidated Guides (Start Here)

| File | Description |
|------|-------------|
| [specs/guides/effect-guide.md](specs/guides/effect-guide.md) | Effect patterns, layers, errors, SQL, testing |
| [specs/guides/testing-guide.md](specs/guides/testing-guide.md) | Unit, integration, E2E testing |
| [specs/guides/frontend-guide.md](specs/guides/frontend-guide.md) | React, UI, components, styling, design system |
| [specs/guides/api-guide.md](specs/guides/api-guide.md) | HttpApi, endpoints, schemas, SSR |

### Architecture

| File | Description |
|------|-------------|
| [specs/architecture/accounting-research.md](specs/architecture/accounting-research.md) | Comprehensive domain spec - US GAAP, entities, services, reports |
| [specs/architecture/authentication.md](specs/architecture/authentication.md) | Multi-provider auth system, session management |
| [specs/architecture/authorization.md](specs/architecture/authorization.md) | RBAC/ABAC policies and permissions |
| [specs/architecture/error-design.md](specs/architecture/error-design.md) | One-layer error architecture |
| [specs/architecture/fiscal-periods.md](specs/architecture/fiscal-periods.md) | Fiscal year/period management, mandatory period 13 |

### Pending Implementation

| File | Description |
|------|-------------|
| [specs/pending/exchange-rate-sync.md](specs/pending/exchange-rate-sync.md) | ECB exchange rate sync and cross-rate triangulation |
| [specs/pending/policy-ux-improvements.md](specs/pending/policy-ux-improvements.md) | Policy creation UX redesign |
| [specs/pending/duplicated-company-creation-page.md](specs/pending/duplicated-company-creation-page.md) | Remove duplicate company creation UI |

### Completed (Historical Reference)

| File | Description |
|------|-------------|
| [specs/completed/consolidated-reports.md](specs/completed/consolidated-reports.md) | Consolidated financial reports |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikearnaldi/accountability](https://github.com/mikearnaldi/accountability) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
