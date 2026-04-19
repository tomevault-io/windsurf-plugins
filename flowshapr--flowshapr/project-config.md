---
trigger: always_on
description: This file provides high-level guidance to Claude Code (claude.ai/code) when working with the Flowshapr monorepo.
---

# CLAUDE.md

This file provides high-level guidance to Claude Code (claude.ai/code) when working with the Flowshapr monorepo.

## Project Overview

Flowshapr is a visual drag-and-drop interface for building Firebase Genkit AI flows. The system allows users to create, manage, and deploy AI flows to various platforms while providing a thin SDK for remote execution. It generates production-ready TypeScript code from visual flows and executes them in real-time.

## Workspace Structure

This monorepo is organized into focused components with dedicated documentation:

```
genkit-builder/
├── frontend/              # Next.js visual flow builder
│   └── CLAUDE.md         # → Frontend-specific guidance
├── server/               # Express.js API backend
│   └── CLAUDE.md         # → Backend-specific guidance
├── sdk/                  # Client SDK for flow execution
│   └── CLAUDE.md         # → SDK-specific guidance
├── docker/              # Container configurations
├── scripts/             # Build and deployment scripts
├── testapps/           # Test applications
└── docs/               # Documentation
```

## Component-Specific Documentation

**When working on specific parts of the system, refer to the component-specific CLAUDE.md files:**

- **[Frontend Development](./frontend/CLAUDE.md)** - React Flow visual editor, Next.js patterns, UI components
- **[Backend Development](./server/CLAUDE.md)** - Domain-driven design, API architecture, database integration
- **[SDK Development](./sdk/CLAUDE.md)** - Client library, Genkit compatibility, TypeScript definitions

## Workspace Commands

```bash
# Development
npm run dev                              # Start frontend development server
npm run build                            # Build all components
npm run lint --workspace=frontend        # Lint specific workspace
npm run type-check --workspace=frontend  # Type-check specific workspace

# Component-specific development
cd frontend && npm run dev               # Frontend development
cd server && npm run dev                 # Backend development
cd sdk && npm run build                  # SDK development

# Database operations (server)
cd server && npm run db:migrate          # Run database migrations
cd server && npm run db:studio           # Open database GUI

# Testing
cd server && npm run test                # Run backend tests
npm run testapps:all                     # Run integration tests
```

## High-Level Architecture

The system follows a modern **three-tier architecture** with clear separation of concerns:

### 1. **Frontend Layer** (Next.js)
- **Visual Flow Builder**: React Flow-based drag-and-drop interface
- **Real-time Code Generation**: Live TypeScript generation from visual flows
- **Execution Interface**: Flow testing and debugging UI
- **Authentication**: Session-based auth with social providers

### 2. **Backend Layer** (Express.js)
- **Domain-Driven Design**: Clean architecture with focused business domains
- **Unified Authentication**: Laravel Sanctum-style session + token auth
- **Flow Execution**: Containerized Genkit flow execution
- **Multi-tenancy**: Organization and team-based access control

### 3. **SDK Layer** (TypeScript Client)
- **Genkit Compatibility**: Drop-in replacement for Genkit client
- **Multiple Environments**: Production, local, and custom instances
- **Streaming Support**: Real-time flow execution with SSE
- **Type Safety**: Full TypeScript support with generics

## Data Flow Overview

```
Visual Editor → Code Generation → Backend API → Container Execution → Results
     ↑                                  ↓
   User UI ←── Real-time Updates ←── Traces & Logs
```

## Cross-Cutting Concerns

### Security Architecture
- **Input Validation**: Zod schemas across all layers
- **Authorization**: Service-level permissions with central abilities
- **Code Execution**: Isolated Docker containers with resource limits
- **API Security**: Rate limiting, CORS, and secure session management

### Database Design
- **PostgreSQL**: Primary database with Drizzle ORM
- **Multi-tenancy**: Organization → Teams → Users → Flows hierarchy
- **Audit Trails**: Execution traces and user activity logging
- **Performance**: Indexed queries and connection pooling

### Deployment Strategy
- **Containerization**: Docker-based deployment with compose files
- **Environment Separation**: Development, staging, and production configs
- **Monitoring**: Health checks, logging, and telemetry collection
- **Scaling**: Horizontal container scaling for execution workloads

## Development Workflow

### Working on Features
1. **Identify the component**: Determine if your work is frontend, backend, or SDK-focused
2. **Read component docs**: Review the specific CLAUDE.md for detailed guidance
3. **Follow established patterns**: Each component has established architectural patterns
4. **Test thoroughly**: Use component-specific testing strategies
5. **Update documentation**: Keep component-specific docs current

### Cross-Component Changes
When changes affect multiple components:
1. **Plan the interface**: Define clear contracts between components
2. **Update sequentially**: Backend → Frontend → SDK (typical order)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/flowshapr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
