---
trigger: always_on
description: Personal finance management app (Microsoft Money replacement). NestJS backend, Next.js frontend, PostgreSQL database, all running in Docker.
---

# Monize

Personal finance management app (Microsoft Money replacement). NestJS backend, Next.js frontend, PostgreSQL database, all running in Docker.

## Tech Stack

| Layer | Tech | Version |
|-------|------|---------|
| Backend | NestJS + TypeORM | 11.x, TS 5.9 |
| Frontend | Next.js (App Router) + React | 16.x, React 19 |
| Database | PostgreSQL | 16 |
| Styling | Tailwind CSS | 4.x |
| State | Zustand (frontend), class-validator DTOs (backend) |
| Forms | react-hook-form + Zod (frontend), class-validator (backend) |
| Auth | JWT + Passport + OIDC + TOTP 2FA |
| AI | Anthropic SDK, OpenAI SDK, Ollama (user-configurable) |
| Testing | Jest (backend), Vitest (frontend), Playwright (e2e) |

## Commands

```bash
# Everything runs in Docker
docker compose -f docker-compose.dev.yml up

# Backend (from /backend)
npm run start:dev          # Dev server with HMR
npm run test               # All tests
npm run test:unit          # Unit tests only
npm run test:cov           # Coverage report (80% minimum)
npm run lint               # ESLint --fix
npm run build              # Production build

# Frontend (from /frontend)
npm run dev                # Dev server (port 3001)
npm run test               # Vitest
npm run test:cov           # Coverage (65% lines minimum)
npm run type-check         # tsc --noEmit
npm run lint

# E2E (from /e2e)
npm run test               # Playwright
```

## Project Structure

```
backend/src/
  {feature}/
    {feature}.module.ts
    {feature}.controller.ts
    {feature}.service.ts
    entities/{entity}.entity.ts
    dto/create-{entity}.dto.ts
    dto/update-{entity}.dto.ts
    {feature}.service.spec.ts
    {feature}.controller.spec.ts
  common/                    # Shared guards, pipes, decorators, utils
  auth/                      # JWT, OIDC, 2FA, PATs, refresh tokens
  ai/                        # AI providers, query engine, MCP server

frontend/src/
  app/                       # Next.js App Router pages
  components/{feature}/      # Feature-organized React components
  lib/                       # API clients (axios), utilities
  hooks/                     # Custom React hooks
  store/                     # Zustand stores (auth, preferences, demo)
  types/                     # Shared TypeScript interfaces

database/
  schema.sql                 # Complete schema (for fresh installs)
  migrations/                # Incremental SQL migrations (auto-applied on startup)
```

## Critical Rules

### Code Organization
- Many small files over few large files (200-400 lines typical, 800 max)
- Organize by feature/domain, not by type
- Always update `database/schema.sql` alongside any migration

### Shared AI tools (AI Assistant + MCP server)
- Every AI tool that reads or aggregates data must share its implementation between the AI Assistant (`backend/src/ai/query/tool-executor.service.ts`) and the MCP server (`backend/src/mcp/tools/*.tool.ts`).
- Put the shared logic on the relevant domain service (e.g., `PortfolioService.getLlmSummary`, `TransactionAnalyticsService.getTransfersByAccount`). The two tool layers become thin adapters that call it.
- Both surfaces must return the same data shape. The AI tool executor wraps it with `{ summary, sources }`; MCP just `toolResult(data)`s it.
- Adding a new AI tool means wiring it into both layers in the same PR -- never ship a tool to only one of the two.

### Code Style
- No emojis in code, comments, or documentation
- Immutability always -- never mutate objects or arrays
- No `console.log` in production code; use NestJS `Logger` class
- Use proxy, not middleware (middleware is deprecated in this project)

### Code Intelligence
Prefer LSP over Grep/Read for code navigation — it's faster, precise, and avoids reading entire files:
- `workspaceSymbol` to find where something is defined
- `findReferences` to see all usages across the codebase
- `goToDefinition` / `goToImplementation` to jump to source
- `hover` for type info without reading the file
    
Use Grep only when LSP isn't available or for text/pattern searches (comments, strings, config).
    
After writing or editing code, check LSP diagnostics and fix errors before proceeding.

### Security (Do Not Regress)
- Parameterized queries only (TypeORM QueryBuilder or parameterized raw SQL). Never interpolate user input into SQL strings
- All controllers use `@UseGuards(AuthGuard('jwt'))` at class level (except health + auth)
- All service methods derive `userId` from JWT (`req.user.id`), never from request params/body
- All path `:id` params use `ParseUUIDPipe`
- DTOs use `whitelist: true` + `forbidNonWhitelisted: true`
- All user-controlled values in HTML email templates must use `escapeHtml()`
- API keys encrypted with AES-256-GCM before storage, never returned to client
- CSRF double-submit cookie pattern is global; use `@SkipCsrf()` only for non-cookie auth (e.g., PAT bearer)

## Backend Patterns

### NestJS Module Pattern

Every feature follows this structure:

```typescript
// controller -- thin, delegates to service
@Controller('feature')
@UseGuards(AuthGuard('jwt'))
export class FeatureController {
  constructor(private featureService: FeatureService) {}

  @Post()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kenlasko/monize](https://github.com/kenlasko/monize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
