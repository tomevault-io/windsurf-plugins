---
trigger: always_on
description: Mini Loan Management System — a demo of AI-native SDLC for the pre-sales engineering team.
---

# CLAUDE.md — Mini Loan Management System

## Project Context

Mini Loan Management System — a demo of AI-native SDLC for the pre-sales engineering team.
Every feature in this repo was built using the process documented in `docs/04-workflows/`.
Goal: show clean architecture + modern tooling + AI-assisted development end-to-end.

## Stack

- **Backend**: ASP.NET Core 9 · EF Core 9 · SQL Server · FluentValidation · AutoMapper · JWT · Serilog
- **Frontend**: Angular 19 · Angular Material · Signals · Standalone Components · pnpm
- **Testing**: xUnit (75% coverage gate) · Jasmine/Karma
- **CI/CD**: GitHub Actions · Claude Sonnet automation
- **Tools**: .NET SDK 9.0.x (global.json) · Node 20.x (Volta) · pnpm 9.x (Volta) · Docker

## Monorepo Structure

See `docs/00-overview/architecture.md` for the full annotated tree.

**Backend pattern**: single project, folder-organised (N-Tier / CrisMAC AXIS).
Call flow: `Controller → Service → Repository → EF Core DbContext`
No cross-folder circular references. Services do not call other services directly — orchestrate via the controller if needed.

**Frontend**: Angular 19 workspace, two projects: `loan-app` (main app) + `shared-lib` (guards, interceptors, models).

## Naming Conventions

### Backend (C#)

- PascalCase: classes, methods, properties, interfaces (`ILoanRepository`)
- camelCase: local variables, parameters
- Async suffix on all async methods (`GetLoanByIdAsync`)
- No MediatR/CQRS — no Commands, Queries, or Handlers

### Frontend (TypeScript/Angular)

- PascalCase: components, services, pipes, classes
- camelCase: methods, properties, variables
- kebab-case: file names, CSS classes, route paths
- Signals: plain noun (`loans`, `currentUser`, `isLoading`)
- Computed signals: derived prefix or adjective (`filteredLoans`, `isAdmin`)
- Services: `[Feature]Service` · Guards: `[Role/Feature]Guard`

## API Design

### HTTP path naming (mandatory, project-agnostic)

All REST URL paths MUST follow these rules, summarized from [Good routes versus bad routes](https://thuongthanhto.medium.com/good-routes-versus-bad-routes-e7e70dfae1bf) (Thuong To, Medium):

1. **Lowercase paths** — path segments use lowercase; separate words with **hyphens** (not underscores, camelCase, or Title Case in the path). Avoid abbreviated segment names; use full, meaningful words. **Path parameter names** use **camelCase** inside `{ }` (e.g. `{loanId}`).
2. **Hierarchy with `/`** — use forward slashes only for parent/child relationships (never backslashes).
3. **Nouns, plural collections** — resource names are **nouns**, not verbs; use **plural** nouns for collections. Express actions with HTTP methods (and bodies) rather than verb-shaped path segments where possible.
4. **No special characters** in paths — avoid punctuation and symbols in fixed segments (documented exceptions only, e.g. multi-value rules in query strings).
5. **No file extensions** in the URI — do not use `.json` / `.xml` on paths; use the `Accept` header or a query parameter for representation format.
6. **Query string for filters and paging** — filtering, sorting, page size, and page number belong in **query parameters**, not extra path segments.
7. **No trailing slash** — do not publish or document paths with a trailing `/`.

### This repository

- Base path: `/api/v1/`
- Auth: Bearer JWT in `Authorization` header
- Success: `{ data: T, message: string, timestamp: string }`
- Errors: ProblemDetails (RFC 7807) — never leak stack traces
- Versioning: URL path (`/api/v1/`, `/api/v2/`)

## Database Conventions

- Table names: PascalCase (`Users`, `Loans`, `RepaymentSchedules`)
- Migrations: numbered prefix — `001_InitialSchema`, `002_AddLoanIndex`
- EF Core Code-First, Fluent API for all configurations (**no DataAnnotations on entities**)
- No raw SQL — EF Core LINQ queries only

## Testing Standards

- Coverage target: **75% (hard gate in CI)**
- Unit test focus: Services, EMI calculation helper, validators
- Test naming: `MethodName_Scenario_ExpectedResult`
- No mocking of EF Core — use InMemory provider for service tests
- Frontend: test services and signal state, not template rendering

## Workflow Orchestration

### Plan Mode (MANDATORY)

Enter plan mode for ANY task with 3+ steps or architectural impact.
If something breaks mid-task: STOP, re-plan, do not push through.
Use plan mode for verification steps too — not just building.

### Subagent Strategy

Offload research and parallel exploration to subagents.
Keep main context window clean — one subagent per concern.
Use Explore subagent for codebase navigation.
Use Plan subagent for architecture decisions.

### Self-Improvement Loop

After ANY user correction: update `tasks/lessons.md` immediately.
Write a rule that prevents the same mistake.
Review `tasks/lessons.md` at the start of each session on this project.

### Verification Before Done

Never mark a task complete without running the code.
Run tests, check build output, confirm correctness.
Ask: "Would a senior engineer approve this?"

## Task Management

- Write plan to `tasks/todo.md` with checkable items before starting
- Check in with user before implementation begins on non-trivial tasks
- Mark items complete as you go

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aditya-shinde-d2k/mini-loan-management-system](https://github.com/aditya-shinde-d2k/mini-loan-management-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
