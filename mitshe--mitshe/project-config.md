---
trigger: always_on
description: > Ten plik jest czytany przez Claude Code jako kontekst projektu.
---

# CLAUDE.md - mitshe

> Ten plik jest czytany przez Claude Code jako kontekst projektu.
> Aktualizuj go gdy zmieniasz architekturę lub dodajesz nowe moduły.

## Quick Reference

```
Nazwa projektu: mitshe
Stack: Next.js 16 + NestJS 11 + TypeScript + Prisma + PostgreSQL/SQLite
Architektura: Hexagonal + CQRS + Event-Driven (Monorepo)
UI: shadcn/ui + Tailwind CSS 4
Auth: Selfhosted (JWT) lub Clerk (optional)
Build: pnpm + Turborepo
```

---

## 1. Struktura Monorepo

```
mitshe/
├── apps/
│   ├── web/                     # Next.js 16 frontend (@mitshe/web)
│   │   ├── src/
│   │   │   ├── app/             # App Router pages
│   │   │   ├── components/      # React components
│   │   │   └── lib/             # Utilities, API client
│   │   └── Dockerfile
│   └── api/                     # NestJS 11 backend (@mitshe/api)
│       ├── src/
│       │   ├── modules/         # Feature modules
│       │   ├── infrastructure/  # Adapters, DB, queues
│       │   └── domain/          # Entities, events
│       ├── prisma/              # Database schema
│       └── Dockerfile
├── packages/
│   └── types/                   # Shared TypeScript types (@mitshe/types)
├── docker/
│   ├── light/                   # All-in-one container (SQLite + Redis)
│   ├── dev/                     # Development infra (PostgreSQL + Redis)
│   └── prod/                    # Production deployment
├── .github/workflows/           # CI/CD pipelines
├── justfile                     # Task runner
├── pnpm-workspace.yaml          # pnpm workspace config
└── turbo.json                   # Turborepo config
```

### Deployment Modes

| Mode | Database | Redis | Auth | Use Case |
|------|----------|-------|------|----------|
| **Light** | SQLite (embedded) | Embedded | Selfhosted | Quick start, single user |
| **Dev** | PostgreSQL (Docker) | Docker | Selfhosted | Local development |
| **Prod** | PostgreSQL (external) | External | Selfhosted or Clerk | Production, multi-user |

### Authentication Modes

| Mode | `AUTH_MODE` | Description |
|------|-------------|-------------|
| **Selfhosted** | `selfhosted` | Email/password JWT auth (default) |
| **Clerk** | `clerk` | Clerk authentication, multi-user, organizations, RBAC |

---

## 2. Komendy (justfile)

```bash
# Pokaż wszystkie komendy
just

# Development
just dev              # Start infra + apps with hot-reload
just infra            # Start only databases
just infra-down       # Stop databases

# Build
just build            # Build all packages
just typecheck        # TypeScript type checking
just lint             # ESLint

# Database
just db-generate      # Generate Prisma client
just db-migrate       # Run migrations
just db-studio        # Open Prisma Studio

# Light Mode (all-in-one)
just light-build      # Build Docker image
just light            # Run container

# Testing
just test             # Run all tests
just check            # Run all quality checks
```

---

## 3. Architektura

### 3.1 Hexagonal (Ports & Adapters)

```
PORTS (interfaces) → apps/api/src/ports/
ADAPTERS (implementations) → apps/api/src/infrastructure/adapters/
```

**Główne porty:**
| Port | Plik | Adaptery |
|------|------|----------|
| `IssueTrackerPort` | `ports/issue-tracker.port.ts` | JiraAdapter, YouTrackAdapter, LinearAdapter |
| `GitProviderPort` | `ports/git-provider.port.ts` | GitLabAdapter, GitHubAdapter |
| `NotificationPort` | `ports/notification.port.ts` | SlackAdapter, TeamsAdapter, EmailAdapter |
| `AIProviderPort` | `ports/ai-provider.port.ts` | ClaudeAPIAdapter, OpenAIAdapter |

### 3.2 CQRS

```
Commands (write) → apps/api/src/application/commands/
Queries (read) → apps/api/src/application/queries/
```

**Naming convention:**
- Command: `CreateTaskCommand`, `ProcessTaskCommand`
- Handler: `CreateTaskHandler`, `ProcessTaskHandler`
- Query: `GetTaskQuery`, `ListTasksQuery`

### 3.3 Event-Driven

```
Events → apps/api/src/domain/events/
Handlers → apps/api/src/application/events/handlers/
```

**Główne eventy:**
- `TaskCreatedEvent` → triggers AI processing queue
- `TaskCompletedEvent` → triggers notifications
- `MergeRequestCreatedEvent` → updates JIRA, notifies Slack
- `TaskFailedEvent` → notifies human intervention needed

### 3.4 Visual Workflow Builder

**Kluczowa funkcjonalność:** Użytkownicy tworzą workflow za pomocą:
1. **GUI** - Drag & drop blocks, visual editor (React Flow)
2. **IaC (Infrastructure as Code)** - YAML/JSON definicje

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│  JIRA   │───▶│   AI    │───▶│ GitLab  │───▶│  Slack  │
│ Trigger │    │ Analyze │    │   MR    │    │ Notify  │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
```

**Node Types:**
- **Triggers:** `trigger:jira:webhook`, `trigger:gitlab:webhook`, `trigger:manual`, `trigger:schedule`
- **AI Agents:** `ai:analyze`, `ai:developer`, `ai:reviewer`, `ai:tester`, `ai:security`
- **Actions:** `action:jira:update`, `action:gitlab:mr`, `action:slack:notify`
- **Control Flow:** `control:condition`, `control:parallel`, `control:loop`

---

## 4. Konwencje Kodu

### 4.1 Nazewnictwo

```typescript
// Pliki
user.entity.ts          // Entity
create-task.command.ts  // Command
task.repository.ts      // Repository
jira.adapter.ts         // Adapter

// Klasy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitshe/mitshe](https://github.com/mitshe/mitshe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
