---
trigger: always_on
description: Autonoma AI is an agentic end-to-end testing platform. Users create and run automated tests for web, iOS, and Android applications using natural language. The system executes tests on real devices/emulators via Playwright (web) and Appium (mobile), with AI models handling element selection, assertions, self-healing, and agentic decision-making.
---

# Autonoma AI - Agentic End-to-End Testing Platform

## Project Overview

Autonoma AI is an agentic end-to-end testing platform. Users create and run automated tests for web, iOS, and Android applications using natural language. The system executes tests on real devices/emulators via Playwright (web) and Appium (mobile), with AI models handling element selection, assertions, self-healing, and agentic decision-making.

The platform is deployed on Kubernetes. Tests are distributed across device-hosting machines with Redis-based device locking for exclusive access.

## Monorepo Structure

```
root/
├── apps/
│   ├── api/              # Hono + tRPC API server (port 4000)
│   │   └── src/
│   │       ├── routers/       # Thin tRPC routers (wiring only)
│   │       └── controllers/   # Business logic (one directory per router)
│   ├── docs/             # Astro Starlight documentation site
│   ├── ui/               # Vite + React 19 SPA (port 3000)
│   ├── engine-web/       # Playwright-based web test execution
│   ├── engine-mobile/    # Appium-based mobile test execution (iOS + Android)
│   └── jobs/
│       ├── reviewer/     # Post-test AI validation using video recordings
│       └── notifier/     # SNS/SQS → Slack/email notifications
├── packages/
│   ├── ai/               # AI primitives: model registry, visual AI, point/object detection
│   ├── db/               # Prisma schema + generated client (PostgreSQL)
│   ├── types/            # Shared Zod schemas, TypeScript types, constants
│   ├── engine/           # Shared engine: execution agent, commands, driver interfaces
│   ├── device-lock/      # Redis-based distributed device locking
│   ├── blacklight/          # Shared UI component library (Radix + Tailwind + CVA)
│   └── utils/            # Logger (Sentry), storage, image, k8s helpers
├── docker/               # Dockerfiles for all services
└── deployment/           # K8s manifests (future)
```

## Development Commands

```bash
pnpm install             # install all dependencies
pnpm db:generate         # generate Prisma client
pnpm build               # build everything (Turborepo)
pnpm typecheck           # tsc --noEmit on all packages
pnpm lint                # eslint on all packages
pnpm test                # vitest on all packages
pnpm dev                 # starts web (3000) and api (4000) concurrently
pnpm db:migrate          # run Prisma migrations
```

### Adding Dependencies

**Always check `pnpm-workspace.yaml` catalog first before adding a dependency.** The catalog defines pinned versions for shared dependencies. When adding a dependency:

1. Check if it already exists in the `catalog:` section of `pnpm-workspace.yaml`.
2. If it does, use `catalog:` as the version specifier in `package.json` instead of a hardcoded version (e.g., `"zod": "catalog:"`).
3. If it doesn't exist in the catalog, consider whether it should be added there (shared across multiple packages) or pinned locally.

```jsonc
// GOOD - uses catalog version
"dependencies": {
  "zod": "catalog:"
}

// BAD - hardcodes version when catalog exists
"dependencies": {
  "zod": "^3.23.0"
}
```

## Code Conventions

### Module System

**ESM-only everywhere.** Every `package.json` has `"type": "module"`. No CommonJS. Non-negotiable.

**Never use `.js` extensions in imports.** All imports must use bare specifiers without file extensions. TypeScript and the bundler resolve modules automatically.

```ts
// GOOD
import { foo } from "./foo";
import { bar } from "@autonoma/types";

// BAD - never add .js to imports
import { foo } from "./foo.js";
```

### TypeScript - Strictest Configuration

All strict flags enabled. Every package extends `tsconfig.base.json`. This includes `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `verbatimModuleSyntax`, etc.

### Classes vs Functions

- **Needs state or dependencies?** Class with constructor injection.
- **Pure logic with no state?** Function file.
- In practice, almost everything is a class.

### Dependency Injection

Plain constructor injection. No DI framework.

```ts
class StepExecutor {
  constructor(
    private readonly engine: Engine,
    private readonly logger: BackendLogger,
  ) {}
}
```

### One Export per File

A file exports exactly one thing. The exported function/class tells the story top-to-bottom. Private helpers follow in call order.

### Database Transactions

**Wrap sequential DB queries in a Prisma `$transaction` when they must be consistent.** If a service method reads then writes (or writes multiple tables), use `db.$transaction(async (tx) => { ... })` and pass `tx` to all queries inside.

```ts
async createGeneration(userId: string, orgId: string, appId: string) {
  return await this.db.$transaction(async (tx) => {
    const app = await tx.application.findFirst({ where: { id: appId, organizationId: orgId } });
    if (app == null) throw new Error("Application not found");

    const generation = await tx.applicationGeneration.create({ data: { ... } });

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Autonoma-AI/autonoma](https://github.com/Autonoma-AI/autonoma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
