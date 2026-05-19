---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants working with this codebase.
---

# CLAUDE.md - AI Assistant Guide

This document provides comprehensive guidance for AI assistants working with this codebase.

## Project Overview

This is a **fullstack monorepo** built with **Turborepo**, featuring a **Next.js 15** frontend and **NestJS** backend with **Prisma ORM**. The architecture follows a **feature-based** pattern, emphasizing maintainability, scalability, and productivity. The entire stack runs in Docker for consistent development environments.

**Key Characteristics:**
- Monorepo with pnpm workspaces
- Feature-based architecture (domain-driven organization)
- TypeScript throughout
- Comprehensive test coverage
- Redis caching layer
- JWT-based authentication with HTTP-only cookies
- SSR with React Query hydration

## Repository Structure

```
/home/user/take-home/
├── apps/
│   ├── backend/          # NestJS API (port 3001)
│   ├── web/              # Next.js frontend (port 3000)
│   └── docs/             # Documentation site
├── packages/
│   ├── ui/               # Shared React components
│   ├── eslint-config/    # ESLint configurations
│   └── typescript-config/# TypeScript configurations
├── docker-compose.yml    # Full stack orchestration
├── turbo.json            # Turborepo configuration
└── .env                  # Docker environment variables
```

### Backend Structure (`apps/backend/`)

```
apps/backend/
├── prisma/
│   ├── schema.prisma     # Database schema (User model)
│   └── seed.ts           # Database seeding script
├── src/
│   ├── auth/             # Authentication module
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   └── auth.guard.ts
│   ├── users/            # Users feature (feature-based)
│   │   ├── controllers/users.controller.ts
│   │   ├── services/users.service.ts
│   │   └── dto/          # Data transfer objects
│   ├── prisma/           # Prisma module (global)
│   │   ├── prisma.module.ts
│   │   └── prisma.service.ts
│   ├── redis/            # Redis caching module
│   │   ├── redis.module.ts
│   │   └── redis.service.ts
│   ├── app.module.ts
│   └── main.ts           # Entry point (port 3000 internal)
├── test/                 # E2E tests
│   ├── *.e2e-spec.ts
│   └── jest-e2e.json
├── Dockerfile
├── init.sh               # Docker initialization script
└── .env.local.example    # Local development env template
```

### Frontend Structure (`apps/web/`)

```
apps/web/
├── app/
│   ├── auth/             # Authentication pages (feature-based)
│   │   ├── login/
│   │   │   ├── page.tsx
│   │   │   └── page.test.tsx
│   │   └── register/
│   │       ├── page.tsx
│   │       └── page.test.tsx
│   ├── users/            # Users feature (feature-based)
│   │   ├── components/
│   │   │   ├── UserList.tsx
│   │   │   ├── UserList.test.tsx
│   │   │   ├── UserForm.tsx
│   │   │   ├── UserForm.test.tsx
│   │   │   ├── UserDeleteButton.tsx
│   │   │   └── UserDeleteButton.test.tsx
│   │   ├── [id]/page.tsx    # Edit user (dynamic route)
│   │   ├── new/page.tsx     # Create user
│   │   └── page.tsx         # Users list (SSR + hydration)
│   ├── api/              # API routes (proxies to backend)
│   │   └── auth/
│   │       ├── login/route.ts
│   │       ├── logout/route.ts
│   │       └── register/route.ts
│   ├── components/       # Shared components
│   │   ├── Header.tsx
│   │   └── LogoutButton.tsx
│   ├── layout.tsx        # Root layout (QueryClientProvider)
│   ├── page.tsx          # Home (redirects to /auth/register)
│   ├── providers.tsx     # React Query setup
│   └── globals.css
├── lib/
│   ├── services/
│   │   ├── userService.ts      # API client functions
│   │   └── userService.test.ts
│   └── queryClient.ts    # React Query client factory
├── middleware.ts         # Route protection
├── Dockerfile
├── jest.config.ts
└── .env.local.example    # Local development env template
```

## Tech Stack

### Backend
- **Framework:** NestJS 11
- **ORM:** Prisma 6.6.0
- **Database:** PostgreSQL 16 (Docker)
- **Cache:** Redis 7.2 (ioredis)
- **Auth:** JWT (jsonwebtoken) + bcrypt
- **Testing:** Jest + Supertest
- **Language:** TypeScript 5.8.2
- **Build:** NestJS CLI with SWC

### Frontend
- **Framework:** Next.js 15.3.0 (App Router)
- **React:** 19.1.0
- **Data Fetching:** TanStack React Query v5.74.4
- **Styling:** Tailwind CSS 3.4.1
- **Testing:** Jest + React Testing Library
- **Build:** Turbopack (dev)
- **Language:** TypeScript 5.8.2

### Infrastructure
- **Monorepo:** Turborepo 2.5.0
- **Package Manager:** pnpm 9.0.0 (required)
- **Containers:** Docker + Docker Compose
- **Node:** 18+ (20-alpine in Docker)

## Development Workflows

### Essential Commands

**From repository root:**

```bash
# Install all dependencies
pnpm install

# Start entire stack (Docker)
docker-compose up -d

# Stop entire stack
docker-compose down

# Development mode (all apps)
pnpm dev

# Build all apps
pnpm build

# Run all linters
pnpm lint

# Format code
pnpm format

# Type check all apps
pnpm check-types
```

**Backend-specific (from root):**

```bash
# Run backend tests (unit + integration)
pnpm --filter backend test

# Run E2E tests (requires Docker running)
pnpm --filter backend test:e2e

# Test with coverage
pnpm --filter backend test:cov

# Generate Prisma client for Docker

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roxdavirox/take-home](https://github.com/roxdavirox/take-home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
