---
trigger: always_on
description: This repository is an Nx monorepo with a NestJS API (`apps/api`) and a React web app (`apps/web`).
---

# UBE-HR Agent Guidelines

This repository is an Nx monorepo with a NestJS API (`apps/api`) and a React web app (`apps/web`).

## Project Structure & Commands

| Project     | Root           | Tech Stack              | Main Commands                                            |
| ----------- | -------------- | ----------------------- | -------------------------------------------------------- |
| **API**     | `apps/api`     | NestJS, Prisma, Webpack | `npx nx build api`, `npx nx test api`, `npx nx lint api` |
| **Web**     | `apps/web`     | React, Vite, Tailwind   | `npx nx build web`, `npx nx test web`, `npx nx lint web` |
| **Feature** | `libs/feature` | NestJS Services         | `npx nx test feature`                                    |
| **Shared**  | `libs/shared`  | TypeScript (Wire Types) | `npx nx test shared`                                     |
| **UI**      | `libs/ui`      | Radix UI, Tailwind      | `npx nx test ui`                                         |
| **Backend** | `libs/backend` | Prisma Service, Config  | `npx nx test backend`                                    |

### Key Development Commands

- **Start All**: `npm run dev` (Starts API on 3000, Web on 4200, and Docker DB)
- **Single Test**: `npx nx test <project> --testFile=<path>`
- **Type Check**: `npx nx run-many -t typecheck` (web) or `npx nx lint`
- **Swagger Docs**: `http://localhost:3000/api/docs`

### Nx Generators

When creating new files, prefer Nx generators over writing files by hand. Always pass `--dry-run` first to preview output, then re-run without it.

**NestJS (`@nx/nest`) — use for `libs/feature` and `apps/api`**

```bash
# Module + Service (new feature in libs/feature)
npx nx g @nx/nest:module  <entity> --project=feature --directory=libs/feature/src/<entity>
npx nx g @nx/nest:service <entity> --project=feature --directory=libs/feature/src/<entity>

# Controller (in apps/api)
npx nx g @nx/nest:controller <entity> --project=api --directory=apps/api/src/app/<entity>

# Other building blocks
npx nx g @nx/nest:guard      <name> --project=feature --directory=libs/feature/src/<entity>/guards
npx nx g @nx/nest:middleware <name> --project=feature --directory=libs/feature/src/<entity>/middleware
npx nx g @nx/nest:decorator  <name> --project=feature --directory=libs/feature/src/<entity>/decorators
npx nx g @nx/nest:interceptor <name> --project=feature --directory=libs/feature/src/<entity>
```

**React (`@nx/react`) — use for `apps/web`**

```bash
# Component
npx nx g @nx/react:component <ComponentName> --project=web --directory=apps/web/src/features/<entity>/components

# Hook
npx nx g @nx/react:hook use<Entity> --project=web --directory=apps/web/src/features/<entity>
```

**Libraries (`@nx/js` / `@nx/react` / `@nx/nest`) — use when adding a new lib**

```bash
# TypeScript lib (e.g. shared, backend)
npx nx g @nx/js:library <lib-name> --directory=libs/<lib-name>

# React component lib (e.g. ui)
npx nx g @nx/react:library <lib-name> --directory=libs/<lib-name>

# NestJS feature lib
npx nx g @nx/nest:library <lib-name> --directory=libs/<lib-name>
```

> After running a generator, review generated files and remove any boilerplate that conflicts with project conventions (e.g. unwanted spec files, default exports, auto-wired module imports that don't match the existing structure).

## Library Architecture

| Import Alias      | Path                | Purpose                                                  |
| ----------------- | ------------------- | -------------------------------------------------------- |
| `@ube-hr/feature` | `libs/feature/src/` | Business logic, services, modules, guards.               |
| `@ube-hr/shared`  | `libs/shared/src/`  | Wire types, permissions, constants (framework-agnostic). |
| `@ube-hr/backend` | `libs/backend/src/` | `PrismaService`, `AppConfigModule`, low-level utils.     |
| `@ube-hr/ui`      | `libs/ui/src/`      | Radix UI + Tailwind design system components.            |

- **Strict Dependency Rule**: `libs/` **cannot** import from `apps/`.
- **Feature Rule**: Use relative imports between sibling modules within `libs/feature`. Use the alias only from outside.
- **Why `libs/backend` is not inside `libs/feature`**: `libs/backend` is pure infrastructure — DB connection (`PrismaService`), env config (`AppConfigModule`), file storage (`StorageService`), and crypto (`secrets`). It has zero knowledge of the HR domain. `libs/feature` depends on it; the reverse is never true. Keeping them separate enforces this one-way dependency, makes infrastructure swappable (e.g. replacing storage backends) without touching business logic, and gives `apps/api`, `apps/worker`, and test helpers a clean place to import low-level primitives without pulling in domain services.

## Key Files by Concern

### Backend

| Concern                                    | File                                                               |
| ------------------------------------------ | ------------------------------------------------------------------ |
| Server bootstrap                           | `apps/api/src/main.ts`                                             |
| Root module (middleware, imports)          | `apps/api/src/app/app.module.ts`                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prymag/ube-hr](https://github.com/prymag/ube-hr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
