---
trigger: always_on
description: Breeze is a fast, modern Remote Monitoring and Management (RMM) platform for MSPs and internal IT teams. Target: 10,000+ agents with enterprise features.
---

# Breeze RMM - Codex Context

## Project Overview

Breeze is a fast, modern Remote Monitoring and Management (RMM) platform for MSPs and internal IT teams. Target: 10,000+ agents with enterprise features.

## Tech Stack

- **Frontend**: Astro + React Islands
- **API**: Hono (TypeScript)
- **Database**: PostgreSQL + Drizzle ORM
- **Queue**: BullMQ + Redis
- **Agent**: Go (cross-platform)
- **Real-time**: HTTP polling + WebSocket
- **Remote Access**: WebRTC

## Key Patterns

### Multi-Tenant Hierarchy
```
Partner (MSP) → Organization (Customer) → Site (Location) → Device Group → Device
```

### Database Schema Location
- `apps/api/src/db/schema/` - All Drizzle schema definitions
- Key tables: devices, users, organizations, sites, alerts, scripts, automations

### API Routes
- `apps/api/src/routes/` - Hono route handlers
- Pattern: Export `xxxRoutes` from each file, mount in `index.ts`

### File Size Guideline
- **Aim to keep files under 500 lines** as a soft guideline, not a hard rule. Use judgment — if a file is cohesive and readable at 600 lines, that's fine. Split when a file becomes hard to navigate or mixes unrelated concerns, not just because it crossed a line count.
- **Declarative files** (e.g. `aiTools*.ts`, schema definitions) can naturally run longer since they're mostly self-contained registration blocks.
- Follow the `aiTools*.ts` pattern: one thin hub file for registry/exports, per-domain files for implementations (e.g. `aiToolsDevice.ts`, `aiToolsNetwork.ts`).
- For route files, split by resource. For service files, split by domain. Helpers used by multiple files can be duplicated locally or extracted to a shared utils file.
- **Do not proactively split files** that are working well just to meet a line count target. Only split when it improves clarity or maintainability.

### Context Preservation
- **Prefer subagents (Agent tool) for research, exploration, and isolated tasks** to keep the main conversation context lean and avoid hitting context limits during long sessions.
- Use subagents for: codebase searches, file reading/analysis, PR reviews, build log inspection, and any work that produces large output.
- Keep the main context for: decision-making, coordinating work, and user interaction.

### Shared Code
- `packages/shared/src/types/` - TypeScript interfaces
- `packages/shared/src/validators/` - Zod schemas
- `packages/shared/src/utils/` - Utility functions

---

## Testing Standards

### Frameworks & Configuration
- **API**: Vitest — `apps/api/vitest.config.ts` (unit), `vitest.config.rls.ts` (RLS), `vitest.integration.config.ts` (integration)
- **Web**: Vitest + jsdom — `apps/web/vitest.config.ts`
- **Agent**: Go standard `testing` package — `go test -race ./...`
- **Shared**: Vitest — `packages/shared/vitest.config.ts`
- **E2E**: Playwright Test (TypeScript), `data-testid` based — `e2e-tests/playwright.config.ts`, specs under `e2e-tests/tests/*.spec.ts`, Page Objects under `e2e-tests/pages/`. Tests query DOM via `data-testid` attributes only (not text/role/CSS) — see `e2e-tests/README.md` for the convention.

### Test File Placement
- Place test files **alongside source files**, not in separate directories
- API: `routes/devices.ts` → `routes/devices.test.ts`
- Go: `internal/discovery/scanner.go` → `internal/discovery/scanner_test.go`
- Shared: `validators/filters.ts` → `validators/filters.test.ts`

### Writing API Route Tests (Vitest)
- Mock Drizzle ORM query chains matching the exact chain pattern in the source (e.g., `select().from().where()`)
- Always test **multi-tenant isolation** — verify org-scoped data can't be accessed cross-org
- Test all HTTP methods, auth/authz, Zod validation failures, not-found, and error cases
- Use proper UUIDs in mock data — Zod validates UUID format and will reject `'other-org'`
- Avoid trailing slashes in test URLs — Hono sub-routers return 404 for trailing slashes
- `vi.mock` factories are hoisted — don't reference module-level `const` values inside them; use literal values instead
- Read 2-3 existing test files in the same directory before writing new ones to match patterns

### Writing Go Agent Tests
- Use **table-driven tests** for functions with multiple input/output combinations
- Always run with `-race` flag to catch data races
- Mock external dependencies (network, OS, filesystem) — never make real network calls
- Use build tags for platform-specific tests: `//go:build !windows` or `//go:build darwin`
- Test nil/empty inputs, error paths, and concurrency safety (spawn goroutines in tests)
- Place test helpers in the same package, not in a separate `_test` package

### Writing Shared Validator Tests
- Test valid inputs, invalid inputs, boundary values, and Zod defaults/coercion
- For discriminated unions, test each variant separately
- Test `omitempty`/optional fields with both present and absent values
- For schemas with `superRefine`, test all validation branches

### What Every New Feature Must Test
1. **Happy path** — basic success case
2. **Auth/authz** — unauthenticated, wrong role, wrong org
3. **Validation** — missing required fields, invalid types, boundary values
4. **Multi-tenant isolation** — cross-org access denied

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LanternOps/breeze](https://github.com/LanternOps/breeze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
