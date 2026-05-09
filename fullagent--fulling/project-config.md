---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

**Fulling v2** is an AI-powered development platform that integrates AI Agent ecosystem to provide full-stack development capabilities. Users can import existing projects from GitHub or create new projects directly on the platform.

**Core Value**: Free users' mental bandwidth through AI Agents. Users focus on development while Agents silently handle complex operations (deployment, infrastructure, etc.) without interruption.

**Key Features**:
- **Flexible Project Creation**: Import from GitHub repositories or create new projects from scratch
- **Optional Database**: Add PostgreSQL database on-demand when needed
- **AI Agent Ecosystem**: AI agents handle development, testing, deployment, and infrastructure management
- **Automated Operations**: Deployment, scaling, and infrastructure management happen automatically in the background
- **Full-Stack Development**: Complete environment with optional database, terminal, and file management
- **Zero Infrastructure Knowledge Required**: Users don't need to understand Kubernetes, networking, or DevOps

**Architecture**: The platform uses an **asynchronous reconciliation pattern** where API endpoints return immediately and background jobs sync desired state (database) with actual state (Kubernetes) every 3 seconds.

## Tech Stack

### Frontend
- Framework: Next.js 16 (App Router) + React 19
- Language: TypeScript
- Styling: Tailwind CSS v4
- UI Components: Shadcn/UI

### Backend
- Runtime: Node.js 22
- API: Next.js API Routes
- Database ORM: Prisma
- Authentication: NextAuth v5 (GitHub, Password, Sealos OAuth)

### Infrastructure
- Container Orchestration: Kubernetes
- Database: PostgreSQL (via KubeBlocks)
- Web Terminal: ttyd (HTTP Basic Auth)
- File Manager: FileBrowser

## Key Conventions

### Code Style
- Use TypeScript strict mode
- Always follow best practices
- Write self-documenting code: for complex functions, describe purpose, expected inputs, and expected outputs above the function
- In `lib/platform/`, prefer one primary action per file. Use noun directories and verb file names, and add a short boundary comment above the main exported function covering purpose, expected inputs/preconditions, expected outputs/guarantees, and what is out of scope.
- Use functional components with hooks

### Naming Conventions
- K8s resources: `{k8s-project-name}-{resource-type}-{6chars}`
- Environment variables: `UPPER_SNAKE_CASE`
- Database tables: PascalCase (Prisma models)
- API routes: kebab-case
- Files: kebab-case
- In `lib/platform/`, file names should usually be action-oriented verbs such as `create-project-with-sandbox`, `find-installation-repository`, or `get-user-default-namespace`, rather than vague names like `shared` or `helpers`.

### Component Organization
- **Route-specific components**: Place in `_components/` directory under the route folder
  - Use `_` prefix to prevent Next.js from treating it as a route
  - Example: `app/(dashboard)/settings/_components/github-status-card.tsx`
- **Shared components**: Place in top-level `components/` directory
  - Only for components used across multiple routes
  - Example: `components/ui/button.tsx`, `components/sidebar.tsx`

### Important Patterns

1. **Always use user-specific K8s service**:
   ```typescript
   const k8sService = await getK8sServiceForUser(userId)
   ```

2. **API endpoints are non-blocking**:
   - Only update database
   - Return immediately
   - Reconciliation jobs handle K8s operations

3. **Use optimistic locking**:
   - Repository layer handles locking automatically
   - Prevents concurrent updates

4. **Follow reconciliation pattern**:
   - API → Database → Reconciliation Job → Event → K8s Operation
   - Status updates happen asynchronously

## Key Design Decisions

### Why StatefulSet?
- Persistent storage for each pod
- Stable network identities
- Ordered pod deployment

### Why Reconciliation Pattern?
- Non-blocking API responses
- Automatic recovery from failures
- Consistent state management
- Easy to monitor and debug

### Why User-Specific Namespaces?
- Multi-tenancy isolation
- Resource quotas per user
- Separate kubeconfig per user
- No cross-user access

## Project Structure

```
Fulling/
├── app/                          # Next.js App Router
│   ├── api/                      # API Routes
│   ├── (auth)/                   # Auth pages
│   └── (dashboard)/projects/[id]/
│       └── _components/          # Route-specific components
│
├── components/                   # Shared components
├── hooks/                        # Client-side hooks
│
├── lib/
│   ├── data/                     # Server-side data access (for Server Components)
│   ├── actions/                  # Client-side data access (Server Actions)
│   ├── repo/                     # Repository layer with optimistic locking
│   ├── services/                 # Business logic services
│   ├── events/                   # Event bus and listeners
│   ├── jobs/                     # Reconciliation background jobs
│   ├── startup/                  # Application initialization
│   ├── k8s/                      # Kubernetes operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FullAgent/fulling](https://github.com/FullAgent/fulling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
