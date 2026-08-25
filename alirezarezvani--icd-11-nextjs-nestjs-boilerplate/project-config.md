---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Architecture

This is a full-stack ICD-11 medical code search application with enterprise-grade authentication:
- **Frontend**: Next.js 13 with TypeScript, Material UI, React Context for state management, next-i18next for internationalization
- **Backend**: NestJS with TypeScript, JWT authentication, TypeORM, Redis caching, Swagger documentation
- **Authentication**: Complete JWT-based auth system with role-based access control (RBAC)
- **External API**: WHO ICD-11 API integration with OAuth2 authentication
- **Caching**: Redis for WHO API response caching with TTL strategies
- **Database**: PostgreSQL with TypeORM for user management and audit trails
- **Internationalization**: Complete i18n support with RTL layout for Arabic language

### Key Modules
- `frontend/`: Next.js application with pages, components, hooks, and services
- `frontend/components/Auth/`: Complete authentication UI components and forms
- `frontend/contexts/`: Authentication context and state management
- `frontend/hooks/`: Authentication hooks and utilities
- `backend/src/auth/`: JWT authentication module with login, register, and token management
- `backend/src/users/`: User entity and management with TypeORM
- `backend/src/icd11/`: Core ICD-11 search functionality and WHO API integration
- `backend/src/cache/`: Redis caching service with cache interface
- `backend/src/common/`: Shared DTOs, interfaces, exceptions, and utilities

## Development Commands

### Root Project Commands
```bash
# Install all dependencies for both frontend and backend
npm run install:all

# Development (runs both frontend and backend concurrently)
npm run dev

# Build both applications
npm run build

# Production start
npm run start
```

### Frontend Commands (from /frontend)
```bash
# Development server (runs on port 3000)
npm run dev

# Production build
npm run build

# Start production server
npm run start

# Linting
npm run lint

# Type checking
npm run type-check

# Code formatting
npm run format
```

### Backend Commands (from /backend) 
```bash
# Development server with watch mode (runs on port 3003)
npm run start:dev

# Production build
npm run build

# Start production server
npm run start:prod

# Linting with auto-fix
npm run lint

# Run tests
npm run test
npm run test:watch
npm run test:cov
npm run test:e2e
```

### Docker Commands
```bash
# Build and start with Docker Compose
npm run docker:up

# Build containers
npm run docker:build

# Stop containers
npm run docker:down
```

## Core Implementation Patterns

### Backend Architecture
- **Modular NestJS structure**: Each feature has its own module (AuthModule, UsersModule, ICD11Module, CacheModule)
- **JWT Authentication**: Access tokens (15 minutes) and refresh tokens (7 days) with automatic rotation
- **Role-Based Access Control**: Multi-tier permissions (USER, HEALTHCARE_PROVIDER, ORG_ADMIN, SUPER_ADMIN)
- **Database Integration**: TypeORM with PostgreSQL for user entities and audit logging
- **Security Features**: Account lockout (5 failed attempts = 30 min), bcrypt password hashing, HIPAA-compliant audit trails
- **Dependency injection**: Services are injected through NestJS DI container
- **DTO validation**: Use class-validator and class-transformer for request/response validation
- **Redis caching**: WHO API responses are cached with appropriate TTL
- **Global Middleware**: Authentication middleware with @Public() decorator for public endpoints
- **Error handling**: Global exception filters and logging interceptors

### Frontend Architecture  
- **Page-based routing**: Next.js file-system routing in `/pages` directory with protected routes
- **Authentication System**: Complete React Context-based auth state management with SSR-safe token storage
- **Component composition**: Reusable components in `/components` with index exports
- **Auth Components**: Login/Register forms with Material-UI and React Hook Form validation
- **Custom hooks**: Business logic encapsulated in `/hooks` (useAuth, useICD11Search, useSearch)
- **API services**: Centralized API client in `/services/api/` with automatic token management
- **Context for state**: React Context for global application state and authentication
- **Protected Routes**: Higher-order components and route guards for role-based access
- **Storage Utilities**: SSR-safe localStorage/sessionStorage with memory fallbacks
- **Internationalization**: next-i18next framework with 6 supported languages (en, es, fr, ar, zh, ru)
- **RTL Support**: Comprehensive right-to-left layout for Arabic with Material-UI theming
- **Translation Management**: Organized namespace structure (common, search, medical, errors, accessibility, auth)

### API Integration
- **WHO ICD-11 API**: OAuth2 authentication with client credentials flow
- **Rate limiting**: Respectful API usage with proper throttling
- **Circuit breaker pattern**: Graceful handling of API failures
- **Response normalization**: Consistent data structures across the application
- **Search Debouncing**: 500ms debounce delay prevents rapid API calls during typing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alirezarezvani/icd-11-nextjs-nestjs-boilerplate](https://github.com/alirezarezvani/icd-11-nextjs-nestjs-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
