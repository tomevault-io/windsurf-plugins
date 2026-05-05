---
trigger: always_on
description: Cognipeer Console is a **multi-tenant SaaS platform** for AI and Agentic services that can operate both as SaaS and on-premise. The platform provides LLM services, agent orchestration, vector stores, workflow automation, and analytics with complete data isolation per company.
---

# Cognipeer Console - Development Guidelines

## Project Overview

Cognipeer Console is a **multi-tenant SaaS platform** for AI and Agentic services that can operate both as SaaS and on-premise. The platform provides LLM services, agent orchestration, vector stores, workflow automation, and analytics with complete data isolation per company.

## Architecture

### Technology Stack

- **Frontend**: Next.js 15 with App Router, TypeScript, Mantine v7 UI
- **Backend**: Next.js API Routes
- **Database**: MongoDB (multi-tenant) with abstraction layer for future provider changes
- **Authentication**: JWT (using jose library for Edge Runtime compatibility)
- **Email**: Nodemailer with Handlebars templates
- **Styling**: Mantine theme system + Tailwind CSS

### Multi-Tenant Architecture

**CRITICAL**: The system is now fully multi-tenant. Each company has:
- A unique slug (URL-friendly identifier)
- A separate MongoDB database for complete data isolation
- License-based feature access

**Database Structure**:
```
MongoDB Server
├── console_main (Main/Shared)
│   └── tenants collection
├── tenant_{slug} (Per Company)
│   ├── users
│   └── api_tokens
└── ...
```

See [MULTI_TENANT_GUIDE.md](../MULTI_TENANT_GUIDE.md) for detailed architecture.

### Key Design Patterns

#### 1. Multi-Tenant Database Abstraction Layer

The database layer supports multi-tenancy with complete isolation:

```typescript
// Location: src/lib/database/
// - provider/contract.ts: Defines DatabaseProvider contract
// - provider/types.base.ts: Core entity interfaces (ITenant, IUser, IApiToken, ...)
// - provider.interface.ts: Backward-compatible re-export for older imports
// - mongodb.provider.ts: Multi-tenant MongoDB implementation
// - index.ts: Database factory with tenant switching

// Usage for tenant operations (uses main DB):
import { getDatabase } from '@/lib/database';
const db = await getDatabase();
const tenant = await db.findTenantBySlug(slug);

// Usage for user/token operations (uses tenant DB):
const db = await getDatabase();
await db.switchToTenant(`tenant_${slug}`);
const user = await db.findUserByEmail(email);
```

**Important**: 
- Main database: Tenant metadata only
- Tenant database: User and API token data
- Always call `switchToTenant()` before user/token operations
- Never import MongoDB directly in application code

#### 2. License-Based Feature Control

Features are controlled through a license system with JWT integration:

```typescript
// Location: src/config/policies.json
// Defines all features and license tiers

// Location: src/lib/license/
// - license-manager.ts: Feature and license utilities
// - token-manager.ts: JWT token management (includes tenant info)

// Usage:
import { LicenseManager } from '@/lib/license/license-manager';
const hasAccess = LicenseManager.hasFeature(licenseType, 'LLM_CHAT');
```

**Flow**:
1. User logs in → JWT generated with license features AND tenant info
2. JWT stored in HTTP-only cookie
3. Middleware checks feature access on each request
4. API routes can access user AND tenant info via headers

#### 3. Middleware-Based Access Control

```typescript
// Location: src/middleware.ts
// - Validates JWT tokens (using jose for Edge Runtime)
// - Checks feature access for API routes
// - Injects user AND tenant info into headers

// Headers available in API routes:
// - x-user-id
// - x-user-email
// - x-user-role
// - x-tenant-id
// - x-tenant-slug
// - x-license-type
// - x-features
```

**Important**: All protected routes automatically go through this middleware. Public paths are defined in the middleware file. Middleware uses `jose` library which is compatible with Edge Runtime.

#### 4. Email System

```typescript
// Location: src/lib/email/mailer.ts
// Handlebars-based template engine

// Templates: mail-templates/*.html
// Template format:
// <!-- subject: Email Subject -->
// <html>...</html>

// Usage:
import { sendEmail } from '@/lib/email/mailer';
await sendEmail(email, 'welcome', { name, companyName, slug, licenseType });
```

## Provider Architecture

### Contract Model

- Contracts live in `src/lib/providers/contracts` and must conform to the `ProviderContract` shape defined in `src/lib/providers/types.ts`.
- Each contract declares its `id`, semantic `version`, supported `domains` (for example, `vector`, `model`), a `display` configuration, optional `capabilities`, a `form` schema, and a `createRuntime` factory that returns a domain-specific runtime.
- Contracts are aggregated in `CORE_PROVIDER_CONTRACTS` (`src/lib/providers/contracts/index.ts`) and auto-registered by the `ProviderRegistry` on first use.

### Provider Registry & Domains

- `ProviderRegistry` (`src/lib/providers/registry.ts`) manages contract registration, descriptor listings, and runtime creation.
- Domain-specific runtime contracts live under `src/lib/providers/domains/*`:
  - `vector.ts` defines the `VectorProviderRuntime` interface (index CRUD, vector upsert/query/delete).
  - `model.ts` defines chat/embedding runtime capabilities and capability flags.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cognipeer/console](https://github.com/Cognipeer/console) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
