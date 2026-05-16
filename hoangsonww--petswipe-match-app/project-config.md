---
trigger: always_on
description: Single source of truth for every AI coding agent working on the PetSwipe project.
---

# AGENTS.md -- PetSwipe Flywheel Operating Manual

Single source of truth for every AI coding agent working on the PetSwipe project.
Read this file at session start and after every context compaction.

---

## Rule 0: Override Prerogative

The human operator's instructions override everything in this file. If a human tells
you to do something that contradicts a rule below, follow the human.

---

## Rule 1: Core Safety Rules

1. **Never delete files** without explicit human permission.
2. **No destructive git commands**: `git reset --hard`, `git clean -fd`, `rm -rf`, `git push --force` are forbidden unless the human explicitly requests them.
3. **All work happens on the main branch** unless the human instructs otherwise.
4. **No script-based code changes**: always edit files manually through your editor tools. Do not generate and execute sed/awk/perl one-liners to modify source code.
5. **No file proliferation**: do not create `mainV2.ts`, `app-backup.ts`, or similar variants. Edit in place.
6. **Compiler and lint checks after every change**: run the appropriate validation command (see Validation Guidance below) before declaring work complete.
7. **Multi-agent awareness**: never stash, revert, or overwrite changes made by other agents. Treat other agents' uncommitted and committed work as your own. If you encounter a merge conflict, stop and notify the human.

---

## Project Overview

PetSwipe is a full-stack pet adoption platform. Users browse adoptable pets via a swipe
interface, match with pets they like, and connect with shelters. The codebase spans a
Next.js frontend, an Express API backend, Kubernetes manifests, Terraform IaC, and
operational scripts.

### Repo Map

| Path | Purpose |
|------|---------|
| `frontend/pages/` | Next.js Pages Router -- main UI routes and API routes |
| `frontend/components/` | Shared layout, Navbar, chart wrapper, shadcn UI primitives |
| `frontend/components/ui/` | 46 shadcn/Radix UI primitives (button, card, dialog, form, table, etc.) |
| `backend/src/routes/` | Express route definitions: auth, chat, matches, pets, swipes, users |
| `backend/src/controllers/` | Request handlers: authController, chatController, matchController, petController, swipeController, userController |
| `backend/src/services/` | Business logic: geocodeService, imageService, userService |
| `backend/src/entities/` | TypeORM entities: Pet, User (AppUser), Match, Swipe |
| `backend/src/middlewares/` | auth.ts (JWT verification), errorHandler.ts |
| `backend/src/app.ts` | Express app setup, route mounting, Swagger, health endpoints |
| `k8s/base/` | Default Kubernetes stack |
| `k8s/overlays/production/` | Production Kustomize overlay |
| `terraform/` | AWS-oriented IaC + optional Vault/Consul/Nomad modules |
| `scripts/` | Deployment and operations scripts |
| `mcp_server/` | MCP (Model Context Protocol) server for AI-assisted features |
| `docs/` | DEPLOYMENT.md, DEVOPS_GUIDE.md, and other operator docs |

---

## Stack and Architecture

### Frontend

- **Framework**: Next.js (Pages Router, not App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Components**: shadcn/ui primitives (Radix UI based), 46 components in `frontend/components/ui/`
- **Animation**: Framer Motion
- **Charts**: Recharts (wrapped in `frontend/components/`)
- **Data fetching**: SWR, Axios
- **Forms**: React Hook Form + Zod validation
- **Theming**: next-themes (light/dark)
- **Icons**: Lucide React

### Backend

- **Framework**: Express.js
- **Language**: TypeScript
- **ORM**: TypeORM
- **Database**: PostgreSQL (Supabase-hosted or AWS RDS)
- **Auth**: JWT bearer tokens -- issued at login, verified by `backend/src/middlewares/auth.ts`
- **API docs**: Swagger UI at `/docs`, OpenAPI JSON at `/api-docs.json`
- **Health**: `/health` (liveness), `/ready` (readiness)
- **File handling**: Multer (upload), Sharp (image processing)
- **Logging**: Morgan

### Auth Flow

1. Client sends credentials to `POST /api/auth/login`.
2. Backend validates and returns a JWT.
3. Client stores the token and sends it as `Authorization: Bearer <token>` on subsequent requests.
4. `authMiddleware` in `backend/src/middlewares/auth.ts` extracts and verifies the token.
5. Verified user ID is attached to `req.user` for downstream handlers.

### API Resource Routes

| Prefix | Router file | Controller |
|--------|-------------|------------|
| `/api/auth` | `routes/auth.ts` | `authController.ts` |
| `/api/pets` | `routes/pets.ts` | `petController.ts` |
| `/api/swipes` | `routes/swipes.ts` | `swipeController.ts` |
| `/api/matches` | `routes/matches.ts` | `matchController.ts` |
| `/api/users` | `routes/users.ts` | `userController.ts` |
| `/api/chat` | `routes/chat.ts` | `chatController.ts` |

### Infrastructure

- **Containers**: Docker (multi-stage builds)
- **Orchestration**: Kubernetes (base + production overlay via Kustomize)
- **IaC**: Terraform targeting AWS (ECS/Fargate, RDS, S3, ALB, CloudFront, WAF)
- **Secrets**: HashiCorp Vault (optional module)
- **Service mesh**: Consul (optional module)
- **CI/CD**: GitHub Actions
- **Monitoring**: Prometheus + Grafana + AWS CloudWatch

---

## Commands That Matter

### Frontend

| Command | Purpose |
|---------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoangsonww/PetSwipe-Match-App](https://github.com/hoangsonww/PetSwipe-Match-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
