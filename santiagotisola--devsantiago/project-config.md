---
trigger: always_on
description: SaaS multi-tenant para gestão de condomínios. O repositório é um monorepo com uma API REST em Node.js/TypeScript, um painel web em React, um app mobile em React e um microsserviço em C# para gestão de encomendas.
---

# CondoSync — Diretrizes do Workspace

SaaS multi-tenant para gestão de condomínios. O repositório é um monorepo com uma API REST em Node.js/TypeScript, um painel web em React, um app mobile em React e um microsserviço em C# para gestão de encomendas.

**Documentação importante**: [README.md](README.md), [HOMOLOGACAO_UNIFICADA.md](HOMOLOGACAO_UNIFICADA.md), [docs/](docs/)

## Arquitetura

```
condosync/
├── apps/api/          # Express + TypeScript + Prisma + PostgreSQL (porta 3333)
├── apps/web/          # React 18 + Vite + Tailwind (porta 5173) — painel admin/funcionários
├── apps/mobile/       # React + Vite + Tailwind + Capacitor — PWA mobile do morador
└── condosync-encomendas/   # Microsserviço ASP.NET Core 10 — fluxo de encomendas
```

**Padrão de módulo da API** (29 módulos: auth, users, condominiums, units, residents, visitors, parcels, vehicles, finance, maintenance, communication, common-areas, digital-signage, documents, employees, service-providers, stock, tickets, pets, renovations, reports, AI, assembly, etc.):
- `{modulo}.routes.ts` — definições de rota + binding ao router
- `{modulo}.controller.ts` — handlers HTTP (thin, delega para service, validação Zod)
- `{modulo}.service.ts` — lógica de negócio, acesso direto ao Prisma client
- DTOs com validação Zod (schemas)

**Arquivos-chave de infraestrutura:**
- Entrada & middlewares: [apps/api/src/server.ts](apps/api/src/server.ts) (Helmet, rate limit, CORS, error handler)
- Schema Prisma: [apps/api/prisma/schema.prisma](apps/api/prisma/schema.prisma) (Unit, Resident, Visitor, Parcel, ServiceOrder, FinancialTransaction, etc.)
- Cliente de API web: [apps/web/src/services/api.ts](apps/web/src/services/api.ts) (axios + interceptores JWT)
- Configuração web: [apps/web/src/vite.config.ts](apps/web/src/vite.config.ts) (proxy dev para API)

## Stack de Tecnologias

| Camada | Tecnologia |
|--------|-----------|
| **Backend API** | Node.js 18+, Express 4.18, TypeScript 5.4, Prisma 5.10 (ORM) |
| **Banco** | PostgreSQL 16, Redis 7 (cache/queue) |
| **Web Frontend** | React 18, Vite, TypeScript, Tailwind CSS, Radix UI, React Query, Zustand |
| **Mobile PWA** | React 18, Vite, Capacitor 7, vite-plugin-pwa, TypeScript, Tailwind |
| **Real-time** | Socket.IO 4.7 (notificações, alertas visitantes, pânico) |
| **Auth** | JWT (1h access + 7d refresh), bcryptjs |
| **Email** | Nodemailer, Resend, Mailpit (dev) |
| **Queue** | BullMQ + Redis |
| **Logging** | Winston, Morgan, Sentry (prod opcional) |
| **Test** | Vitest (API), Playwright (E2E), xUnit (C#) |
| **Security** | Helmet, express-rate-limit, CORS, xss-pkg |
| **Microsserviço** | ASP.NET Core 10, Entity Framework 9, SQL Server |

## Build & Testes

```Padrões & Convenções

**API (Node.js/Express)**
- **Auth**: JWT access token (1h) + refresh token (7d); papéis: `SUPER_ADMIN`, `CONDOMINIUM_ADMIN`, `SYNDIC`, `DOORMAN`, `RESIDENT`, `SERVICE_PROVIDER`, `COUNCIL_MEMBER`.
- **Imports**: Use o alias `@/` para `apps/api/src/`.
- **Tratamento de erros**: `express-async-errors` captura rejeições de promises; lance erros tipados nos services, middleware central `errorHandler` formata respostas HTTP.
- **Validação**: Zod DTOs nos controllers para validar input antes de delegar ao service.
- **Rate limiting & CORS**: Já configurados em [server.ts](apps/api/src/server.ts) — não adicione guards redundantes.
- **Queue jobs**: Use BullMQ + Redis para tarefas assíncronas (email, webhooks, processamento pesado).
- **Timeout real-time**: Socket.IO sem persistência — conexões perdidas = eventos perdidos.

**Web (React/Zustand/React Query)**
- **Estado global**: Zustand (persistido em localStorage) para auth/UI.
- **Dados servidor**: React Query para fetch/cache/sync automático.
- **Imports**: Use o alias `@/` para `apps/web/src/`.
- **Proxy dev**: Vite proxeia http://localhost:3333 para `localhost:5173/api/`.

**Mobile (React PWA/Capacitor)**
- **Build**: `npm run android:apk` (debug), `npm run android:apk:release` (release) — requer Windows PowerShell.
- **PWA**: Offline-first com service worker + cache strategy.

**Database (Prisma/PostgreSQL)**
- **Seed**: `seed-demo.js` (dados realistas), `seed-base.js` (fixtures mínimas).
- **Migration**: `npx prisma migrate dev --name <descricao>` (dev), `npx prisma migrate deploy` (prod).
- **Gateways de pagamento**: ASAAS, PJBANK integrado
npx prisma generate     # Gera cliente Prisma
npx prisma migrate dev  # Dev migrations
npm run db:seed         # ts-node prisma/seed.ts (dados demo realistas)

# Web & Mobile (similarmente, com npm run dev, build, test)
cd apps/web && npm run dev    # http://localhost:5173
cd apps/mobile && npm run dev # http://localhost:5174
Microsserviço C# (Encomendas)

Localização: `condosync-encomendas/`
- **Framework**: ASP.NET Core 10, Entity Framework 9
- **Banco**: SQL Server (não PostgreSQL)
- **Padrão**: Controllers → Services → Repositories
- **Testes**: xUnit em `tests/` (EncomendaServiceTests, MoradorServiceTests)

## Deploy

**Local**
- Docker Compose: `docker compose up` → postgres, redis, api (3333), web (80), mobile (80 alt port)
- Credenciais Postgres: `condosync/condosync123`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [santiagotisola/DevSantiago](https://github.com/santiagotisola/DevSantiago) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
