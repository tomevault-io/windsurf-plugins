---
trigger: always_on
description: Adaptive full-stack system design skill for Node.js backends integrated with React, Next.js, and Vite, supporting monorepo/polyrepo architectures, multiple testing tools, and scalable production systems. Acts as a senior architect that guides developers through key decisions before generating tailored, production-ready code.
---


# Full-Stack Node.js System Design

A smart, adaptive skill that behaves like a principal architect: it gathers your context first, then produces the right structure, patterns, and code for your specific project — not a one-size-fits-all template.

---

## 1. When to Use This Skill

- Starting any new full-stack project with a Node.js backend
- Choosing between monorepo and polyrepo architecture
- Integrating React (Vite), Next.js (App Router), or both with a backend
- Establishing auth, API contracts, testing, and deployment strategies
- Onboarding a team to consistent, scalable conventions
- Migrating a legacy project to a modern full-stack setup

---

## 2. Interactive Architecture Detection

**Before generating any structure or code, ask the user these questions.**

```
1. What are you building?
   [ ] SaaS product        [ ] Internal dashboard
   [ ] Public API           [ ] Real-time application
   [ ] Microservices system [ ] Not sure yet

2. What frontend framework will you use?
   [ ] React (Vite)         [ ] Next.js (App Router)
   [ ] Both                 [ ] Not decided

3. Architecture preference?
   [ ] Monorepo             [ ] Polyrepo
   [ ] Not sure — recommend based on my scale

4. Team / project scale?
   [ ] Solo / hobby         [ ] Small team (2–5)
   [ ] Startup (5–20)       [ ] Enterprise / large team

5. Primary database?
   [ ] PostgreSQL / MySQL (SQL)
   [ ] MongoDB (NoSQL)
   [ ] Both
   [ ] Undecided

6. Testing preference?
   [ ] Jest                 [ ] Vitest
   [ ] Mocha + Chai         [ ] Playwright (E2E)
   [ ] Cypress (E2E)        [ ] Not sure
```

Use answers to drive every decision below. Revisit defaults only when answers are "not sure."

---

## 3. Architecture Decision Engine

### Monorepo vs Polyrepo

**Recommend Monorepo when:**
- Frontend and backend share TypeScript types (DTOs, schemas)
- One team maintains both layers
- You use Next.js (naturally co-located with its API routes)
- You want a single CI/CD pipeline
- Project scale: solo → startup

**Recommend Polyrepo when:**
- Independent teams own each service
- Services have different release cycles
- You are building true microservices with separate CI/CD
- Scale: large-scale enterprise

**Always explain the trade-off:**

> Monorepo = faster DX, shared types, simpler CI. Cost: repo grows with the project.
> Polyrepo = independent deployability, cleaner ownership. Cost: type duplication, complex CI coordination.

---

## 4. Project Structures

### Option A — Monorepo (Turborepo + pnpm)

**When to use:** Shared types matter. One team. Next.js or React + backend together.

```
apps/
├── web/              # React (Vite) or Next.js frontend
├── admin/            # Optional internal dashboard
└── api/              # Express or Fastify backend

packages/
├── shared/           # DTOs, Zod schemas, ApiResponse types
├── ui/               # Shared component library
├── api-client/       # Typed fetch/axios SDK
└── config/           # ESLint, TSConfig, env schemas

turbo.json
pnpm-workspace.yaml
```

```yaml
# pnpm-workspace.yaml
packages:
  - "apps/*"
  - "packages/*"
```

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": { "dependsOn": ["^build"], "outputs": ["dist/**"] },
    "dev": { "cache": false, "persistent": true },
    "test": { "dependsOn": ["build"] },
    "lint": {}
  }
}
```

### Option B — Polyrepo

**When to use:** Independent teams. Different deployment cycles.

```
backend/          # Node.js API (standalone repo)
frontend/         # React or Next.js (standalone repo)
shared-types/     # Optional: published npm package for DTOs
```

Each repo has its own `package.json`, CI pipeline, and deploy target. Sync types via a versioned npm package (`@myorg/shared@1.2.0`).

### Option C — Microservices

**When to use:** High scale, independent scaling requirements, distributed teams.

```
services/
├── gateway/          # API gateway (routes + auth checks)
├── auth-service/     # JWT issuance, refresh, user identity
├── user-service/     # User CRUD
└── notification-service/

packages/
├── shared/           # Shared event types, error classes
└── api-client/       # Internal service clients

infra/
├── docker-compose.yml
└── nginx/
```

> For microservices, each service is an independent Fastify/Express app. Services communicate via HTTP or a message broker (Redis Pub/Sub, RabbitMQ). The gateway is the sole public entry point.

---

## 5. Backend System

### 5.1 Base Setup (Express or Fastify)

**Express:**

```typescript
// apps/api/src/app.ts
import express from "express";
import helmet from "helmet";
import { corsMiddleware } from "./config/cors";
import { sanitize } from "./middleware/sanitize";
import { requestLogger } from "./middleware/logger";
import { errorHandler } from "./middleware/error-handler";
import { apiV1Router } from "./api/v1/routes";

const app = express();

app.use(helmet());
app.use(corsMiddleware);
app.use(express.json({ limit: "10mb" }));
app.use(sanitize);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ameriq8/fullstack-nodejs-system-design](https://github.com/Ameriq8/fullstack-nodejs-system-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
