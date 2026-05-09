---
trigger: always_on
description: Enterprise-grade platform for managing AI-powered conversations at scale.
---

# Chat Platform - AI Chat Management System

Enterprise-grade platform for managing AI-powered conversations at scale.

## Project Overview

See @README.md for detailed product information
See @package.json for scripts and dependencies
See @src/CLAUDE.md for source code architecture

## Quick Start

```bash
# Install dependencies
npm install

# Configure environment
cp .env.example .env
# Edit .env with your credentials

# Setup database (Local Development)
supabase start               # Start local Supabase
supabase db reset            # Apply migrations

# Setup database (Production)
supabase link --project-ref your-project-ref
supabase db push             # Push migrations to remote

# Start development
npm run dev

# Run tests
npm test
```

## Project Structure

```
chat_platform/
├── src/              # Source code (@src/CLAUDE.md)
├── public/           # Static assets
├── docs/             # Documentation (@docs/CLAUDE.md)
├── tools/            # Development tools (@tools/CLAUDE.md)
├── .env.example      # Environment template
├── package.json      # Dependencies & scripts
└── README.md         # Product overview
```

## Key Technologies

- **Next.js 14** - Full-stack React framework
- **TypeScript** - Type safety throughout
- **PostgreSQL** - Relational database (via Supabase)
- **Supabase** - Database, auth, and real-time subscriptions
- **Supabase Auth** - Authentication & user management
- **Tailwind CSS** - Utility-first styling
- **Row-Level Security (RLS)** - Centralized authorization system

## Development Commands

```bash
# Core Development
npm run dev                  # Start dev server (port 3000)
npm run build               # Production build
npm run start               # Production server
npm run type-check          # TypeScript validation
npm run lint                # ESLint checks
npm run lint:fix            # Auto-fix issues

# Database Management (Supabase)
supabase migration new <name>  # Create new migration
npm run db:reset               # Reset local database
npm run db:push                # Push migrations to remote
npm run db:pull                # Pull schema from remote
npm run db:types               # Generate TypeScript types
npm run db:studio              # Open Supabase Studio
npm run db:seed                # Seed test data

# Testing
npm test                   # Run all tests
npm run test:watch         # Watch mode
npm run test:coverage      # Coverage report
npm run test:ui           # Test UI components

# Utilities
npm run clean:cache        # Clear Next.js cache
npm run analyze           # Bundle analysis
npm run tunnel            # Webhook tunnel (dev)
```

## Architecture Principles

### Multi-Tenant Design

- Organization-based data isolation
- **Row-Level Security (RLS)**: Centralized authorization with database-agnostic tenant isolation
- Scalable permission system with role-based access control
- Complete audit trail for all data access

### Security First

- **RLS-Protected Database Access**: All queries automatically filtered by tenant
- Authentication required on all routes
- Input validation on every endpoint
- Encrypted sensitive data storage
- Rate limiting on API calls
- Comprehensive audit logging

### Performance Optimized

- Server-side rendering where appropriate
- Optimistic UI updates
- Efficient database queries
- Response caching strategies

### Developer Experience

- Full TypeScript coverage
- Comprehensive error messages
- Hot module replacement
- Detailed logging in development

## Code Organization Conventions

### Directory Structure Philosophy

We follow a **feature-first** organization pattern with clear separation of concerns:

```
src/
├── app/              # Next.js App Router - routing only
├── components/
│   ├── ui/          # Design system primitives
│   ├── shared/      # Reusable cross-feature components
│   ├── features/    # Feature-specific components
│   └── layout/      # Page structure components
├── lib/             # Core infrastructure & integrations
├── utils/           # Pure utility functions
├── actions/         # Server actions (mutations)
├── hooks/           # React custom hooks
└── types/           # Shared TypeScript types
```

### lib/ vs utils/ vs helpers/

- **`lib/`** - Core infrastructure, third-party integrations, complex business logic
  - Examples: Database clients, auth providers, AI integrations
  - Typically has side effects or external dependencies

- **`utils/`** - Pure functions, formatters, validators
  - Examples: Date formatting, string manipulation, data transformation
  - Should be side-effect free and easily testable
  - Organized by domain: `formatters/`, `validators/`, `parsers/`

- **No `helpers/` directory** - Avoid generic catch-all folders
  - Name by domain instead (formatters, validators, parsers)

### Component Organization

- **`components/ui/`** - Design system primitives (Button, Input, Dialog)
- **`components/shared/`** - Reusable composite components (BaseTable, EmptyState)
- **`components/features/`** - Feature-specific components with business logic
- **`components/layout/`** - Page structure components (Header, Sidebar, Layout)

### Colocation Principles

Keep related files together:

```
ComponentName/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [connorbell133/Agentflow](https://github.com/connorbell133/Agentflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
