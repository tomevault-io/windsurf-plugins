---
trigger: always_on
description: Role: Senior Fullstack Engineer (NestJS & Next.js expert)
---

# Project Context: Vully - Apartment Management Platform

Role: Senior Fullstack Engineer (NestJS & Next.js expert)

## Tech Stack

### Backend
- **Framework**: NestJS (Modular Architecture)
- **Database**: PostgreSQL 15+ with Prisma ORM
- **Cache/Queue**: Redis 7+ with BullMQ
- **Real-time**: Socket.IO WebSocket Gateway
- **AI**: LangChain.js with pgvector for RAG
- **Logging**: Pino (structured JSON logging)
- **File Validation**: ClamAV integration + MIME type checking
- **Health Checks**: @nestjs/terminus (DB, Redis, Queue health)
- **Validation**: Zod (shared schemas via packages/shared-types)
- **Docs**: Swagger/OpenAPI auto-generated

### Frontend
- **Framework**: Next.js 15 (App Router)
- **UI Components**: Shadcn/UI (strictly no native HTML for buttons/inputs/modals)
- **State**: TanStack Query (server state) + Zustand (global state) + Nuqs (URL state)
- **Forms**: React-Hook-Form + Zod
- **Tables**: TanStack Table (with virtualization for >100 rows)
- **Styling**: Tailwind CSS
- **Animations**: Framer Motion (page transitions, layout animations)
- **Onboarding**: Shepherd.js (interactive walkthroughs)
- **Charts**: Recharts
- **Maps**: Custom SVG with D3.js (optional)

## Architecture Guidelines

### Backend (NestJS)
- Always use **Modular Architecture** - each feature is a self-contained module
- Every module MUST have: `*.module.ts`, `*.controller.ts`, `*.service.ts`, `*.dto.ts`
- Use **Global Exception Filter** (`HttpExceptionFilter`) for standardized error responses
- Use **Logging Interceptor** for performance metrics and audit trails with correlation IDs
- Background jobs MUST use BullMQ queues (never block main thread, retry 3 times with exponential backoff)
- Database operations MUST use Prisma transactions where appropriate
- WebSocket events MUST use Socket.IO gateway with room-based broadcasting (building, apartment, user scopes)

**Implemented Modules (✅)**:
- Identity: Auth (JWT access+refresh, password reset), Users, Multi-role RBAC (UserRoleAssignment + Permissions + RolePermission)
- Apartments: Buildings (with SVG maps + policies), Apartments (50+ fields), Contracts (CRUD + terminate), Access Cards (issue/edit/deactivate/reactivate), Parking (zones + slots), Building Policies (versioned), Payment Schedules
- Payment Tracking: Contract Payment Schedules, Payments, Financial Summaries, Void Support
- Billing: Invoices, Meter Readings, Utility Types/Tiers (tiered pricing), BullMQ processor, Billing Jobs, Management Fee Configs
- Incidents: CRUD, Comments, WebSocket Gateway
- Stats: Dashboard analytics (Redis-cached, 5-min TTL)
- AI Assistant: RAG chatbot (Gemini + pgvector + LangChain), Documents, Document Chunks

**Skeleton Modules (🚧)**: Management Board (empty controllers, need implementation)

**Planned Modules (📋)**:
- Multi-Tenant: Organization, OrganizationMember, PostgreSQL RLS
- Trust Accounting: FinancialAccount, EscrowLedger, EscrowTransaction
- Regional Compliance: ComplianceRule, ComplianceAlert
- Payment Gateway: PaymentIntent, PaymentWebhookLog (Stripe/VNPay/MoMo)
- Communication Hub: NotificationTemplate, NotificationDelivery, UserNotificationPreference

### Frontend (Next.js)
- Use **App Router** with Server Components by default
- Client Components only when needed (interactivity, hooks, state)
- **Always prefer Shadcn/UI components over native HTML** (buttons, inputs, selects, modals, etc.)
- **Use Framer Motion** for any element entering or leaving the DOM (page transitions, list animations)
- Use Framer Motion `LayoutGroup` for smooth list add/remove transitions with `layout` prop
- **Skeleton Loaders** required for all async data (CLS = 0, no layout shift)
- Use `dynamic()` import for heavy widgets (charts, maps, 3D viewer) with skeleton loading state
- **TanStack Table** with windowing/virtualization for lists > 100 items
- **TanStack Query** over useState/useEffect for ALL API calls (never use raw fetch)
- **Nuqs** for URL state management (filters, pagination, tabs) — keeps URL in sync with UI state
- **Zustand** for global client state (authStore, mapStore)

### API Design
- Follow OpenSpec definitions in `openspec/specs/`
- RESTful endpoints with proper HTTP methods
- Consistent response format: `{ data, meta, errors }`
- Pagination: cursor-based for large datasets

## Coding Standards

### TypeScript
- **Strict Mode** enabled (`strict: true` in tsconfig)
- No `any` type - use `unknown` with type guards
- Prefer `interface` for objects, `type` for unions/primitives
- Use `readonly` for immutable data

### Backend Specifics
- DTOs MUST use `class-validator` decorators (@IsString, @IsEmail, @IsOptional, @IsEnum, etc.)
- All Controllers MUST have `@nestjs/swagger` decorators (@ApiTags, @ApiOperation, @ApiResponse, @ApiBearerAuth)
- Services should be stateless and injectable (use Prisma, Redis, BullMQ via injection)
- Use custom decorators for common patterns (`@CurrentUser`, `@Roles`) — already implemented
- **Use Pino logger** for structured logging (JSON format, correlation IDs via CorrelationIdMiddleware)
- **File uploads** MUST validate MIME type and size before processing (ClamAV integration available)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tommyle1310) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
