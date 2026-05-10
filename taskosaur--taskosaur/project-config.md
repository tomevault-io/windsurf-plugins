---
trigger: always_on
description: This document provides guidance for AI agents working with the Taskosaur project, including development setup, testing, and code quality standards.
---

# AGENTS.md - Taskosaur AI Agent Configuration

This document provides guidance for AI agents working with the Taskosaur project, including development setup, testing, and code quality standards.

## Project Overview

Taskosaur is an open-source project management platform with conversational AI task execution. It's built as a monorepo with:
- **Backend**: NestJS API server (port 3000)
- **Frontend**: Next.js application (port 3001)
- **Database**: PostgreSQL with Prisma ORM
- **Queue**: Redis with BullMQ

## Development Setup

### Prerequisites
- Node.js 22+ and npm 10+
- PostgreSQL 16+ (or Docker)
- Redis 7+ (or Docker)

### Quick Start with Docker (Recommended)
```bash
# 1. Copy environment template
cp .env.example .env

# 2. Start all services
docker-compose -f docker-compose.dev.yml up
```

### Manual Setup
```bash
# 1. Install dependencies
npm install

# 2. Setup database
npm run db:migrate
npm run db:seed

# 3. Start development servers
npm run dev
```

## Available Commands

### Development
```bash
npm run dev              # Start both frontend and backend
npm run dev:frontend     # Start frontend only (port 3001)
npm run dev:backend      # Start backend only (port 3000)
```

### Database Operations
```bash
npm run db:migrate       # Run database migrations
npm run db:seed          # Seed database with sample data
npm run db:seed:admin    # Seed database with admin user only
npm run db:reset         # Reset database (deletes all data!)
npm run db:studio        # Open Prisma Studio (database GUI)
```

### Testing
```bash
npm run test             # Run all tests
npm run test:frontend    # Run frontend tests
npm run test:backend     # Run backend unit tests
npm run test:e2e         # Run backend end-to-end tests
npm run test:cov         # Run backend tests with coverage
```

### Code Quality
```bash
npm run lint             # Lint all workspaces
npm run lint:frontend    # Lint frontend code
npm run lint:backend     # Lint backend code
npm run format           # Format backend code with Prettier
```

### Build
```bash
npm run build            # Build all workspaces
npm run build:frontend   # Build frontend for production
npm run build:backend    # Build backend for production
npm run build:dist       # Build complete distribution package
```

### Cleanup
```bash
npm run clean            # Clean all build artifacts
npm run clean:frontend   # Clean frontend build artifacts
npm run clean:backend    # Clean backend build artifacts
```

## Project Structure

```
taskosaur/
├── backend/                # NestJS Backend (Port 3000)
│   ├── src/
│   │   ├── modules/       # Feature modules (auth, tasks, projects, etc.)
│   │   ├── common/        # Shared utilities and middleware
│   │   ├── config/        # Configuration files
│   │   ├── gateway/       # WebSocket gateway
│   │   ├── seeder/        # Database seeding
│   │   └── prisma/        # Database service
│   ├── prisma/            # Database schema and migrations
│   ├── public/            # Static files
│   └── uploads/           # File uploads
├── frontend/              # Next.js Frontend (Port 3001)
│   ├── src/
│   │   ├── app/          # App Router pages
│   │   ├── components/   # React components
│   │   ├── contexts/     # React contexts
│   │   ├── hooks/        # Custom hooks
│   │   ├── lib/          # Utilities
│   │   └── types/        # TypeScript types
│   └── public/           # Static assets
├── docker/               # Docker configuration
├── scripts/              # Build and utility scripts
├── plans/               # Development plans and documentation
└── package.json         # Root package configuration
```

## Coding Standards

### TypeScript & JavaScript
- Use TypeScript for all new code
- Enable strict mode with null checks
- Provide explicit type annotations
- Avoid `any` type unless absolutely necessary

### Backend (NestJS)
- Use dependency injection via constructors
- Implement proper error handling with try/catch
- Use DTOs for request/response validation
- Follow existing module structure

### Frontend (Next.js/React)
- Use functional components with hooks
- Implement proper error boundaries
- Use TypeScript interfaces for props
- Follow existing component structure

### Database (Prisma)
- Write clear, descriptive migration names
- Include both up and down migrations
- Test migrations on sample data
- Document schema changes

## Git Hooks & Code Quality

Automatic code quality checks with **Husky**:
- **Pre-commit**: Runs linters on all workspaces before each commit
- Ensures code quality and consistency
- Bypass with `--no-verify` (emergencies only)

```bash
git commit -m "feat: add feature"  # Runs checks automatically
```

## Testing Guidelines

### Backend Tests
- Unit tests in `backend/test/` directory
- E2E tests in `backend/test/e2e/` directory
- Use Jest as the testing framework
- Mock external dependencies appropriately

### Frontend Tests
- Use Vitest for unit testing
- Use Playwright for E2E testing
- Test components in isolation
- Mock API calls appropriately

## Environment Variables

Key environment variables to configure:

```env
# Database Configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Taskosaur/Taskosaur](https://github.com/Taskosaur/Taskosaur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
