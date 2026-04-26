---
trigger: always_on
description: Main cursor rules for GraphQL Auth project. Read this first!
---

# Main Project Guide

This file provides a comprehensive overview of the project, including architecture, commands, and key development patterns.

## Quick Command Reference

```bash
# Development
bun run dev                             # Start dev server (port 4000)
bun run test --run                      # Run all tests once (no watch)
bun run test                            # Run tests in watch mode
bun test test/auth.test.ts              # Run specific test file
bun run test -t "test name"             # Run tests matching pattern
bun run test:ui                         # Run tests with UI
bun run test:coverage                   # Run tests with coverage

# Database
bunx prisma migrate dev --name feature  # Create migration
bun run generate                        # Generate all types (Prisma + GraphQL)
bun run db:reset                        # Reset database with seed data
bunx prisma studio                      # Open database GUI

# Build & Production
bun run build                          # Build for production (builds src/main.ts)
bun run start                          # Start production server (runs dist/app/server.js)
bun run clean                          # Clean build directory

# Code Quality
bunx tsc --noEmit                      # Type check all files
bunx prettier --write .                # Format code

# GraphQL Schema
bun run gen:schema                      # Generate GraphQL schema file
bunx gql.tada generate-output           # Generate GraphQL type definitions

# Environment & Debugging
bun run env:verify                      # Verify environment setup
bun run graphql:examples                # Run GraphQL query examples
```

## Architecture Overview

### Tech Stack

- **Runtime**: Bun (fast JavaScript/TypeScript runtime)
- **GraphQL Server**: Apollo Server 4 with H3 HTTP framework
- **Schema Builder**: Pothos with advanced plugin integration:
  - **Prisma Plugin**: Direct access pattern (NOT in context) for better TypeScript performance
  - **Relay Plugin**: Global IDs, connections with metadata, cursor pagination
  - **Errors Plugin**: Union result types for comprehensive error handling
  - **Scope Auth Plugin**: Dynamic authorization with 14+ scope types
  - **DataLoader Plugin**: Automatic batch loading for N+1 prevention
  - **Validation Plugin**: Zod integration with async refinements
- **Database**: Prisma ORM with SQLite (dev.db) - accessed directly, not through context
- **Authentication**: JWT tokens with bcryptjs + refresh token rotation
- **Authorization**: GraphQL Shield middleware with Pothos Scope Auth
- **Type Safety**: GraphQL Tada for compile-time GraphQL typing
- **Testing**: Vitest with comprehensive test utilities
- **Validation**: Zod schema validation with custom async refinements
- **Rate Limiting**: rate-limiter-flexible with configurable presets
- **Dependency Injection**: TSyringe for service management

### Architecture: Modular Direct Resolvers

The project uses a **modular architecture** with direct Pothos resolvers organized by feature:

```
src/
├── app/                              # Application entry and configuration
│   ├── [server.ts](mdc:src/app/server.ts)                     # Main server setup (entry point: bun run dev)
│   ├── config/                       # Configuration
│   │   ├── [container.ts](mdc:src/app/config/container.ts)              # TSyringe DI container setup
│   │   ├── [environment.ts](mdc:src/app/config/environment.ts)            # Environment variable handling
│   │   └── [database.ts](mdc:src/app/config/database.ts)               # Database configuration
│   └── middleware/                   # Application-level middleware
│       ├── [cors.ts](mdc:src/app/middleware/cors.ts)                   # CORS configuration
│       ├── [rate-limiting.ts](mdc:src/app/middleware/rate-limiting.ts)          # Rate limiter setup
│       └── [logging.ts](mdc:src/app/middleware/logging.ts)                # Request/response logging
│
├── modules/                          # Feature modules (domain-driven)
│   ├── auth/                         # Authentication module
│   │   ├── [auth.schema.ts](mdc:src/modules/auth/auth.schema.ts)            # GraphQL schema definitions
│   │   ├── [auth.permissions.ts](mdc:src/modules/auth/auth.permissions.ts)       # Authorization rules
│   │   ├── [auth.validation.ts](mdc:src/modules/auth/auth.validation.ts)        # Input validation schemas
│   │   ├── resolvers/                # GraphQL resolvers
│   │   │   ├── [auth.resolver.ts](mdc:src/modules/auth/resolvers/auth.resolver.ts)      # Basic auth (signup/login/me)
│   │   │   └── [auth-tokens.resolver.ts](mdc:src/modules/auth/resolvers/auth-tokens.resolver.ts) # Refresh token operations
│   │   ├── services/                 # Business logic services
│   │   │   ├── [password.service.ts](mdc:src/modules/auth/services/password.service.ts)   # Password hashing (bcryptjs)
│   │   │   └── [token.service.ts](mdc:src/modules/auth/services/token.service.ts)      # JWT token management
│   │   └── types/                    # Module-specific types
│   ├── posts/                        # Posts module
│   │   ├── [posts.schema.ts](mdc:src/modules/posts/posts.schema.ts)           # Post type definitions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/semyenov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
