---
trigger: always_on
description: Stack: NestJS (Node) + .NET 8 + React 18 + PostgreSQL 15 + Redis 7
---

# WIA-272: Billing Links

Stack: NestJS (Node) + .NET 8 + React 18 + PostgreSQL 15 + Redis 7

## Comandos essenciais
- Testes Node: `cd node-api && npm test`
- Testes .NET: `cd dotnet-service-tests && dotnet test`
- Testes Frontend: `cd frontend && npm test`
- Subir infra: `docker compose up -d`

## Regras criticas
- name/cpf NUNCA em logs — PiiSanitizer obrigatorio
- seller_id sempre do JWT
- Idempotency-Key obrigatorio no endpoint publico

<!-- GSD:project-start source:PROJECT.md -->
## Project

**WIA-272: Billing Links — Production Refactor**

Refatorar o PoC de Billing Links para production-ready. O PoC atual demonstra o fluxo completo (seller cria link, cliente paga via endpoint publico, transacao processada pelo servico .NET), mas usa stack desatualizada (Nest 10 + TypeORM), estrutura fragmentada (3 repositorios separados), e tem gaps criticos de funcionalidade. O objetivo e migrar para a stack real da WiteTec (Nest 11 + Prisma), reorganizar em monorepo src/modules, e completar todos os fluxos para producao.

**Core Value:** Seller cria um billing link e compartilha; cliente acessa `/pay/:slug`, paga via PIX ou cartao, e o seller ve o resultado no dashboard em tempo real.

### Constraints

- **Stack:** Nest 11 + Prisma (alinhamento com WiteTec real) — migrar de Nest 10 + TypeORM
- **Seguranca:** name/cpf NUNCA em logs — PiiSanitizer obrigatorio em todos os paths de erro
- **Autenticacao:** seller_id SEMPRE do JWT token (claim `sub`), nunca do body/query
- **Idempotencia:** Idempotency-Key obrigatorio no endpoint publico, implementacao atomica com SET NX
- **Infraestrutura:** PostgreSQL 15 + Redis 7 (ja existem via Docker Compose)
- **Integracao:** .NET service mantido como PSP gateway — comunicacao via HTTP interno
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- TypeScript 5.3.3 - Node.js API (`node-api/`) and Frontend
- C# 8.0 - .NET service (`dotnet-service/`)
- JavaScript/JSX - React components
- SQL - PostgreSQL schema and migrations
## Runtime
- Node.js (inferred 18.x or 20.x from package.json constraints)
- .NET 8.0 runtime
- npm (Node.js projects)
- NuGet (dotnet dependencies)
- Lockfile: `node-api/package-lock.json` and `frontend/package-lock.json` (standard npm lockfiles)
## Frameworks
- NestJS 10.0.0 - Node.js backend API framework (`node-api/`)
- React 18.2.0 - Frontend UI library (`frontend/`)
- ASP.NET Core 8.0 - .NET Web API framework (`dotnet-service/`)
- Jest 29.7.0 - Node.js unit and integration tests (config: `node-api/jest.config.js`)
- Vitest 1.1.0 - Frontend unit tests (config: `frontend/vite.config.ts`)
- xUnit 2.6.2 - .NET unit tests (config: `dotnet-service-tests/WitetecBillingService.Tests.csproj`)
- Vite 5.0.8 - Frontend bundler and dev server (`frontend/`)
- TypeScript 5.3.3 - TypeScript compiler for both Node and frontend
- ts-jest 29.1.1 - Jest transformer for TypeScript
- ts-node-dev 2.0.0 - Development server for Node API
## Key Dependencies
- `@nestjs/core` 10.0.0 - NestJS core framework
- `@nestjs/jwt` 10.0.0 - JWT token handling
- `@nestjs/passport` 10.0.0 - Authentication middleware
- `@nestjs/typeorm` 10.0.0 - ORM integration
- `typeorm` 0.3.17 - SQL ORM with TypeScript support
- `pg` 8.11.3 - PostgreSQL driver
- `ioredis` 5.3.2 - Redis client library
- `axios` 1.6.0 - HTTP client for .NET service communication
- `passport-jwt` 4.0.1 - JWT strategy for Passport
- `class-validator` 0.14.0 - DTO validation
- `class-transformer` 0.5.1 - DTO transformation
- `uuid` 9.0.0 - UUID generation
- `rxjs` 7.8.1 - Reactive programming
- `react-router-dom` 6.21.0 - Client-side routing
- `axios` 1.6.0 - HTTP client for API calls
- `uuid` 9.0.0 - UUID generation
- `clsx` 2.0.0 - Conditional className builder
- `tailwindcss` 3.4.0 - Utility-first CSS framework
- `autoprefixer` 10.4.16 - PostCSS plugin for vendor prefixes
- `Microsoft.EntityFrameworkCore.Design` 8.0.0 - EF Core design-time tools
- `Npgsql.EntityFrameworkCore.PostgreSQL` 8.0.0 - PostgreSQL provider for EF Core
- `Microsoft.AspNetCore.Authentication.JwtBearer` 8.0.0 - JWT authentication
- `Serilog.AspNetCore` 8.0.0 - Structured logging
- `Serilog.Sinks.Console` 5.0.0 - Serilog console output
## Configuration
- Environment variables via `.env` file (see `.env.example` for template)
- Key configs:
- Node.js: `node-api/tsconfig.json` - TypeScript compilation settings
- Frontend: `frontend/tsconfig.json`, `frontend/vite.config.ts` - Build and TypeScript config
- .NET: `dotnet-service/WitetecBillingService.csproj` - Project definition
## Platform Requirements
- Node.js 18.x or higher (implied by package.json)
- .NET SDK 8.0
- PostgreSQL 15 (via Docker)
- Redis 7 (via Docker)
- Docker and Docker Compose
- Containerized deployment (Docker Compose available)
- PostgreSQL 15 instance
- Redis 7 instance
- Node.js runtime for API
- .NET 8.0 runtime for service
- PostgreSQL 15-alpine (from `docker-compose.yml`)
- Connection via `pg` driver (Node) and `Npgsql` (.NET)
- Migrations in `db/migrations/` applied on container startup
- Redis 7-alpine (from `docker-compose.yml`)
- Client: `ioredis` 5.3.2 on Node.js
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Naming Patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cigano-agi/features-wite-tec](https://github.com/Cigano-agi/features-wite-tec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
