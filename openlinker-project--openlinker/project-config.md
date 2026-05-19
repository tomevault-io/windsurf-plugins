---
trigger: always_on
description: @docs/architecture-overview.md
---

@docs/architecture-overview.md
@docs/engineering-standards.md

# OpenLinker – Claude Code Guide

OpenLinker is an open-source, modular, API-first e-commerce orchestration platform built on **Hexagonal Architecture** (Ports and Adapters) in a pnpm monorepo.

---

## Reference Documentation

Before writing or modifying code, read the relevant doc(s):

| Topic | File |
|---|---|
| System architecture, layers, bounded contexts | `docs/architecture-overview.md` |
| Coding standards, naming, file structure | `docs/engineering-standards.md` |
| Testing standards and practices | `docs/testing-guide.md` |
| Code review standards | `docs/code-review-guide.md` |
| Database migration workflow | `docs/migrations.md` |
| Frontend technical architecture and state rules | `docs/frontend-architecture.md` |
| Frontend visual and interaction style | `docs/frontend-ui-style-guide.md` |
| Implementation plan process | `docs/implementation-plan-generator-guide.md` |

Architecture docs define **intent and direction**, not every implementation detail. You may infer missing layers or patterns if they clearly align — but always justify them explicitly.

---

## Architecture Rules

### Backend

The system follows Hexagonal Architecture organized by layer:

- **CORE** (`libs/core/src/`) — domain logic, ports (interfaces), core services. Platform-agnostic.
- **Integrations** (`libs/integrations/`) — adapters implementing CORE ports for external platforms (Allegro, PrestaShop, etc.)
- **Infrastructure** — persistence (PostgreSQL via TypeORM), Redis, repositories
- **Interface** — REST controllers, request/response DTOs, event handlers
- **Shared** (`libs/shared/`) — cross-cutting utilities and types

**CORE vs Integration boundary is strict:**
- CORE defines capability ports (`ProductMasterPort`, `InventoryMasterPort`, `OrderProcessorManagerPort`, etc.)
- Integrations implement those ports — they never bleed domain logic back into CORE
- Do not blur this boundary for convenience

### Frontend

The frontend lives in `apps/web` and is a browser-first admin SPA. See `docs/frontend-architecture.md` for all conventions.

Dependency direction is enforced:
- `app` → `pages` → `features` → `shared`
- `shared` must not import `features` or `pages`

State ownership:
- Server state → TanStack Query
- URL state → route params / search params
- Form state → React Hook Form
- Session state → `SessionProvider`
- Local UI state → component-local `useState` / `useReducer`
- No general-purpose global store for FE-001

---

## Naming Conventions (Summary)

See `docs/engineering-standards.md` for the full list. Key patterns:

**Backend:**
- Ports: `*.port.ts` → class `{Capability}Port`
- Adapters: `*-adapter.ts` → class `{Platform}{Capability}Adapter`
- Services: `*.service.ts` implementing `*.service.interface.ts`
- Entities: `*.entity.ts`, ORM entities: `*.orm-entity.ts`
- Mappers: `*.mapper.ts`, DTOs: `*.dto.ts`
- Unit tests: `*.spec.ts`, integration tests: `*.int-spec.ts`

**Frontend:**
- Components: `PascalCase.tsx`
- Hooks: `use-*.ts`
- Route modules: `*.route.tsx`
- Tests: `*.test.tsx`

---

## Behavior Rules

- **Do not** redefine or reinterpret architecture — follow what the docs say
- **Do not** duplicate architectural explanations in code or comments
- **Do not** bypass CORE ↔ Integration boundaries
- **Prefer** reusing existing abstractions over creating new ones
- **Default** to MVP-appropriate solutions unless explicitly told otherwise
- **Justify** any new pattern, abstraction, or component explicitly

## Code Quality Rules

### Never do these

- No `any` in TypeScript — use proper types or `unknown` with narrowing
- No `console.log` — use the shared `Logger` from `libs/shared/src/logging/`
- No comments that explain *what* code does — comments explain *why*
- No hardcoded secrets, tokens, or credentials anywhere in code
- No `.env` files committed to git
- No `synchronize: true` in TypeORM config — migrations are the source of truth
- No force-push to `main`
- No skipping hooks with `--no-verify`
- No `// eslint-disable` without a specific reason in the same comment

### Security baselines

- Validate all user input at system boundaries (controllers, DTOs with class-validator)
- Use TypeORM query builder — never interpolate user input into raw SQL
- Never return secrets, tokens, or credentials in API responses
- Never embed secrets in frontend code — only `VITE_*` public build-time vars in browser code
- Use `@UseGuards(JwtAuthGuard)` on all non-public endpoints

### TypeScript

- Strict mode is on — no implicit `any`, no loose null checks
- Prefer explicit return types on public methods and exported functions
- Use `unknown` over `any` when type is genuinely unknown; narrow with type guards
- Keep types close to where they are used — colocate with the module that owns them

---

## Working Process

### For code tasks

1. Read relevant documentation and find similar existing patterns in the codebase
2. Propose what will be implemented now vs. what can be deferred
3. Implement conservatively — only what is required, keep changes localized
4. Add or update tests for non-trivial logic
5. Run the quality gate before committing (see below)
6. Commit on a dedicated branch named after the issue (e.g. `55-100-fe-api-client-and-query-layer`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openlinker-project/openlinker](https://github.com/openlinker-project/openlinker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
