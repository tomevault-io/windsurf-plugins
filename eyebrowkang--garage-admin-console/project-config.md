---
trigger: always_on
description: Always use Context7 MCP when I need library/API documentation, code generation, setup or configuration steps without me having to explicitly ask.
---

# AGENTS.md

Always use Context7 MCP when I need library/API documentation, code generation, setup or configuration steps without me having to explicitly ask.

## Project Overview

Garage Admin Console — a web-based administration interface for managing [Garage](https://garagehq.deuxfleurs.fr/) object storage clusters.

## Commands

```bash
pnpm install              # Install dependencies (run pnpm approve-builds if native builds are blocked)
pnpm dev                  # Start both API (port 3001) and web (Vite dev server) concurrently
pnpm build                # Build both packages (api then web)
pnpm lint                 # Lint both packages
pnpm lint:fix             # Auto-fix lint issues
pnpm format               # Format all code with Prettier
pnpm format:check         # Check formatting without writing

# Run commands for a single package:
pnpm -C api <script>      # e.g. pnpm -C api dev, pnpm -C api typecheck
pnpm -C web <script>      # e.g. pnpm -C web dev, pnpm -C web build

# Type checking (api only):
pnpm -C api typecheck     # tsc --noEmit

# Testing:
pnpm -C web test          # Run unit tests in watch mode (Vitest)
pnpm -C web test:run      # Run unit tests once
npx playwright test       # Run E2E tests (Playwright)
```

## Architecture

**Monorepo** (pnpm workspace) with two packages:

- **`api/`** — Backend-For-Frontend (BFF) service: Express 5, Drizzle ORM (SQLite via LibSQL), TypeScript
- **`web/`** — Frontend SPA: React 19, Vite, TanStack React Query, Tailwind CSS, TypeScript

### Data Flow (BFF Proxy Pattern)

The frontend never talks to Garage clusters directly. All Garage API calls go through the BFF proxy:

```
Browser → /proxy/:clusterId/* → BFF decrypts stored admin token → forwards to Garage cluster endpoint
```

### API Routes (`api/src/routes/`)

Routes are registered in `api/src/app.ts`.

| Route                     | Auth | Purpose                         |
| ------------------------- | ---- | ------------------------------- |
| `POST /auth/login`        | No   | Returns JWT                     |
| `GET /health`             | No   | Health check                    |
| `GET /clusters`           | JWT  | List clusters (tokens excluded) |
| `POST /clusters`          | JWT  | Add cluster                     |
| `PUT /clusters/:id`       | JWT  | Update cluster                  |
| `DELETE /clusters/:id`    | JWT  | Remove cluster                  |
| `ALL /proxy/:clusterId/*` | JWT  | Proxy to Garage admin API       |

### Frontend Structure (`web/src/`)

- **Routing** (React Router v7, defined in `App.tsx`): `/login`, `/` (Dashboard), `/clusters/:id` (ClusterDetail with sidebar navigation)
- **Pages**: `Login`, `Dashboard` (cluster list/management), Cluster pages (Overview, Buckets, Keys, Nodes, Layout, Admin Tokens, Workers, Blocks, Metrics)
- **API client** (`lib/api.ts`): Axios instance with JWT interceptor; 401/403 → redirect to login
- **UI**: shadcn/ui components (`components/ui/`) built on Radix UI primitives, styled with Tailwind
- **Path alias**: `@` → `web/src/`

### Database Schema (`api/src/db/schema.ts`)

Two tables defined with Drizzle ORM: `Cluster` (id, name, endpoint, adminToken encrypted, metricToken encrypted optional, timestamps) and `AppSettings` (key-value). Migrations are in `api/drizzle/` and run automatically on startup.

## Frontend UX/UI design principles and approach

### UX

The overall approach progresses from simple to complex, layer by layer.

The outermost layer is the Dashboard page, which displays information for all clusters.
There will be multiple clusters here, usually independent of each other, so it should present key cluster-level information, such as the number of nodes, whether the cluster is healthy, and whether storage capacity is under pressure.

After clicking on a cluster, you enter the operation page for a single cluster.
The overall layout has a function selection area on the left and a function display/operation area on the right.
The first and default function is the Overview page, which displays information about a single cluster.
The information here should be more detailed than on the Dashboard, presenting an overview of a single cluster, including health status, layout information, statistical information, and so on.

Next are seven major functional modules: Buckets, Access Keys, Layout, Nodes, Admin Tokens, Workers, and Blocks.
Each module continues to follow the logic of progressing from simple to complex, layer by layer. For example, in Buckets, the main page displays a list that only shows the primary information about buckets, along with functions to create and delete them.
After clicking on a bucket, you enter the detail page, where you can see more information and perform more operations, such as Aliases and Website Access.

### UI

The theme color is orange `rgb(255, 148, 41)`. The logo comes from Garage official assets and is located in the `web/public` directory.
The overall style is light-themed, and switching to a dark theme is not supported for now.
The page should not use too many colors; keep the style simple.
Red represents errors, green represents health, purple represents warnings.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eyebrowkang/garage-admin-console](https://github.com/eyebrowkang/garage-admin-console) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
