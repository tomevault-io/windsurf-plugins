---
trigger: always_on
description: **Engineer**: Zawadi MC Nyachiya
---

# Cursor Config

**Engineer**: Zawadi MC Nyachiya  
**Last Updated**: 2026-02-15  
**Scope**: Global configuration for all projects

---

## 1. Core Engineering Identity

### Technical Stack Foundations

**Scope**: Global Engineering Standards (encompassing Enterprise & SaaS)

**Primary Languages**:

- **TypeScript** (Strict, Shared Types across Web/Mobile/Edge)
- **C#** (Enterprise Backend Systems)
- **Python** (Cloud Functions for AI/Heavy Lifting)
- **SQL** (PostgreSQL via Drizzle/EF Core)

**Backend Architectures**:

- **Option A (SaaS/Startup)**: **Supabase Serverless** (Auth, Database, Realtime, Edge Functions)
- **Option B (Enterprise)**: **ASP.NET Core (.NET 8+)** + Entity Framework Core
- **Option C (Event-Driven)**: Node.js/Python Cloud Functions (GCP/Azure)
- **Legacy**: Node.js/Express (Deprecated)

**Frontend Ecosystem**:

- **Web**: React + Vite (SPA)
- **Mobile**: React Native + Expo (Managed Workflow)
- **State Management**: Zustand (Client), TanStack Query (Server), Nanostores
- **Styling**: NativeWind (Tailwind CSS v4) or twrnd
- **Motion**: Reanimated (Mobile), Framer Motion (Web)

**Infrastructure & "External Brain"**:

- **Platforms**: Supabase, Google Cloud Platform (GCP), Azure App Services
- **Databases**: Supabase PostgreSQL (RLS Enforced), Firestore
- **Queues**: Upstash QStash (Serverless), Google Pub/Sub
- **Compute**: Supabase Edge Functions (Deno), Google Cloud Functions (Python)

### Architectural Philosophy

Build production-grade systems with clear separation of concerns (Thick Client + Serverless):

- **Domain layer remains pure**: Zero framework dependencies, isolated business logic in `packages/shared-types`
- **Security boundaries are explicit**: RLS (Row Level Security) is the primary firewall
- **Observability is first-class**: Sentry for error tracking, PostHog for analytics
- **Cost optimization matters**: Scale-to-zero infrastructure (Serverless/Edge)

---

## 2. AI Agent Collaboration Protocol

### Read-First, Verify-Always Approach

**Before any code modification**:

1. Read relevant source files using `view_file` or equivalent
2. Check project-specific `agents.md` (if present) for tactical constraints
3. Review existing architecture documentation (`docs/spec/`, `docs/architecture/`, `docs/`)
4. Understand the change's impact on domain boundaries and security posture
5. **Verify edge cases against Production Readiness Framework (Section 9)**

**Never**:

- Run destructive commands without explicit user approval
- Introduce framework dependencies into domain layers
- Skip test execution after logic changes
- Assume implicit requirements—ask for clarification
- Proceed without considering the 28 critical production areas (Section 9)

### Communication Standards

- **Blocked on ambiguity**: Use `notify_user` with `BlockedOnUser: true` immediately
- **Phase completion**: Request explicit review before proceeding to next phase
- **Tradeoff decisions**: Present architectural options with clear pros/cons, cost implications, security considerations, and impact on production readiness areas
- **Error reporting**: Provide specific error messages, file paths, suggested fixes, and impact assessment
- **Edge case identification**: Explicitly call out edge cases discovered during implementation

### Tool Usage Discipline

- **Git**: User handles commits unless explicitly delegated; ensure atomic commits with descriptive messages
- **Builds**: Verify backend changes with `dotnet build` and `dotnet test`; frontend changes with `npm run build`
- **Commands**: Prefer showing commands over executing them—let user maintain control of their environment
- **Documentation**: Update `planning_log.md` or equivalent at phase boundaries
- **Testing**: Run comprehensive test suites before claiming completion

### Coding Conventions

- **File Naming**: strictly use `kebab-case` for all files and directories (e.g., `user-profile.tsx`, `auth-service.ts`)
- **Component Naming**: use `PascalCase` for React components (e.g., `UserProfile`)
- **Documentation**: All exported components and functions must have JSDoc comments explaining their purpose, props, and return values.
- **Imports**: Use relative imports for internal files unless an alias is configured. Ensure import paths match the file system casing exactly.

---

## 3. Code Quality Expectations

### Architecture Boundaries

**Clean Architecture enforcement**:

- Domain entities contain business logic only (no infrastructure leakage)
- Application layer handles use cases and orchestration
- Infrastructure layer owns database, external APIs, cloud services
- API/Presentation layer manages HTTP contracts and DTO transformations

**Dependency Rules**:

- Domain → Zero external dependencies
- Application → Domain only
- Infrastructure → Application + Domain
- API/Presentation → All layers

### Security Posture

**All systems must**:

- Treat AI/LLM output as untrusted input requiring validation
- Enforce authorization at database layer (Firestore rules, row-level security)
- Use Secret Manager/Key Vault for credentials—never hardcode secrets
- Implement rate limiting and quota enforcement for expensive operations
- Validate input schemas strictly before processing
- Sanitize all user-generated content before storage or display

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Z3DDIEZ) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
