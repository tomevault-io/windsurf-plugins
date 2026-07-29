---
trigger: always_on
description: OpenSyte is an open-source all-in-one business management software built with the T3 stack (Next.js, TypeScript, tRPC, Prisma). It provides comprehensive modules for CRM, Project Management, Finance, HR, and Workflow Automation.
---

---
applyTo: "**"
---

## Project Overview

OpenSyte is an open-source all-in-one business management software built with the T3 stack (Next.js, TypeScript, tRPC, Prisma). It provides comprehensive modules for CRM, Project Management, Finance, HR, and Workflow Automation.

## Key Development Commands

### Setup & Installation
```bash
# Install dependencies
bun install

# Set up environment
cp .env.example .env

# Set up database (Linux/macOS)
./start-database.sh

# Set up database schema
bun run db:push

# Generate Prisma client and zod schemas
bun run postinstall
```

### Development
```bash
# Start development server with Turbo
bun run dev

# Type checking
bun run typecheck

# Lint code
bun run lint

# Fix linting issues
bun run lint:fix

# Check code format
bun run format:check

# Format code
bun run format:write

# Full code check (lint + typecheck)
bun run check
```

### Database Operations
```bash
# Generate and apply migrations in development
bun run db:generate

# Deploy migrations to production
bun run db:migrate

# Push schema changes without migrations
bun run db:push

# Open Prisma Studio database GUI
bun run db:studio
```

### Testing & Building
```bash
# Build for production
bun run build

# Start production server
bun run start

# Preview build locally
bun run preview
```

### Shadcn UI Components
```bash
# Add new shadcn/ui components
bun run shadcn add <component-name>
```

## Architecture Overview

### Core Technology Stack
- **Framework**: Next.js 15 with App Router
- **Language**: TypeScript with strict typing
- **Database**: PostgreSQL with Prisma ORM
- **API**: tRPC for type-safe API routes
- **Authentication**: Better Auth with Google OAuth
- **UI**: Tailwind CSS + shadcn/ui components
- **State Management**: Jotai for global state, React Query for server state
- **Runtime**: Bun for package management and development

### Directory Structure
```
src/
├── app/                    # Next.js App Router pages
│   ├── (auth)/            # Authentication routes
│   ├── (dashboard)/       # Main application routes
│   └── accept-invite/     # Invitation acceptance flow
├── components/            # Reusable UI components
│   ├── ui/               # shadcn/ui base components
│   ├── auth/             # Authentication components
│   ├── dashboard/        # Dashboard-specific components
│   └── [module]/         # Module-specific components (CRM, HR, etc.)
├── server/
│   ├── api/              # tRPC API implementation
│   │   └── routers/      # Feature-specific routers
│   └── db.ts             # Database connection
├── lib/                  # Utility functions and configurations
└── styles/               # Global CSS styles
```

### Module Architecture

The application is organized into distinct business modules:

#### 1. CRM Module (`src/server/api/routers/crm/`)
- **Contacts**: Customer and lead management
- **Deals**: Sales pipeline tracking with stages
- **Interactions**: Customer communication history

#### 2. Project Management (`src/server/api/routers/projects/`)
- **Projects**: Project creation and management
- **Tasks**: Task management with Kanban and Gantt views
- Hierarchical task structure with subtasks
- Resource allocation and time tracking

#### 3. Finance Module (`src/server/api/routers/finance/`)
- **Invoices**: Invoice generation and management
- **Expenses**: Expense tracking with categories
- **Financial Reports**: Dynamic report generation engine

#### 4. HR Module (`src/server/api/routers/hr/`)
- Employee management and profiles
- Payroll processing and calculations
- Time-off request management
- Performance review system

#### 5. Workflow Automation (`src/server/api/routers/workflows/`)
- Visual workflow designer using React Flow
- Trigger-based automation system
- Integration with external services (Email, SMS, Calendar)
- Variable system for dynamic data handling

### Authentication & Authorization

#### Multi-Tenant Architecture
- Organization-based multi-tenancy
- User-organization relationships with roles
- Invitation-based user onboarding

#### Role-Based Access Control (RBAC)
- **Predefined Roles**: Organization Owner, Super Admin, Department Managers, etc.
- **Custom Roles**: Organization-specific custom roles with granular permissions
- **Permission System**: Module-based permissions (CRM, Finance, HR, Projects)
- **Context-Aware Authorization**: tRPC procedures with permission middleware

#### Key RBAC Components
- `src/lib/rbac.ts`: Predefined role permissions and metadata
- `src/lib/custom-rbac.ts`: Custom role permission checking
- `src/server/api/trpc.ts`: Permission-based procedure middlewares

### Database Design

#### Core Principles
- **Multi-tenant**: All entities scoped to organizations
- **Audit Trail**: CreatedAt/UpdatedAt timestamps on all entities
- **Soft References**: Strategic use of optional foreign keys
- **JSON Fields**: Flexible configuration storage for workflows and reports


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Opensyte/opensyte](https://github.com/Opensyte/opensyte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
