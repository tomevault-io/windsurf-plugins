---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

analog-tools is a collection of utilities and libraries for [AnalogJS](https://analogjs.org) fullstack meta-framework applications. The project is an Nx monorepo providing authentication, session management, logging, and dependency injection capabilities.

**⚠️ Early Development Stage**: Breaking changes may happen frequently as APIs evolve.

## Available Packages

- `@analog-tools/auth` - OAuth 2.0/OIDC authentication for server-side (h3/Nitro)
- `@analog-tools/auth-angular` - Angular client-side authentication components and services
- `@analog-tools/session` - Session management with pluggable storage backends
- `@analog-tools/inject` - Custom dependency injection framework
- `@analog-tools/logger` - Structured logging with deduplication
- `@analog-tools/generator` - Nx code generators for scaffolding

## Common Commands

### Development

```bash
# Serve the example application
pnpm nx serve analog-example

# Build a specific package
pnpm nx build @analog-tools/auth
pnpm nx build @analog-tools/session

# Build all packages
pnpm nx run-many -t build
```

### Testing

```bash
# Test a specific package
pnpm nx test @analog-tools/auth
pnpm nx test @analog-tools/session

# Test with coverage
pnpm nx test @analog-tools/logger --coverage

# Run all tests
pnpm nx run-many -t test
```

### Package Management

```bash
# Create a package tarball for local testing
pnpm nx pack @analog-tools/auth

# Release workflow (versions and publishes)
pnpm nx release
```

## Architecture

### Package Relationships

```
@analog-tools/auth (server)
    ├── @analog-tools/session
    ├── @analog-tools/inject
    └── @analog-tools/logger

@analog-tools/auth-angular (client)
    └── integrates with @analog-tools/auth

@analog-tools/session
    └── @analog-tools/inject

@analog-tools/logger
    └── @analog-tools/inject
```

### Custom Dependency Injection System

Location: `/packages/inject/src`

The project uses a custom DI system based on a service registry pattern:

- Services must be marked with `static readonly INJECTABLE = true`
- Use `registerService(TokenClass, ...constructorArgs)` to register
- Use `inject(TokenClass)` to retrieve instances
- Singleton pattern: one instance per token
- No decorators or metadata reflection

```typescript
// Mark a service as injectable
class MyService {
  static readonly INJECTABLE = true;
  constructor(config: Config) {}
}

// Register with arguments
registerService(MyService, configObject);

// Inject anywhere
const service = inject(MyService);
```

### Authentication Architecture (BFF Pattern)

Location: `/packages/auth/src/server`

Implements Backend-for-Frontend OAuth 2.0/OIDC authentication:

**Key Components:**

- `useAnalogAuth()` - Initializes auth system and returns route handler
- `OAuthAuthenticationService` - Core OAuth logic (token exchange, refresh, validation)
- `SessionService` - Manages auth sessions via `@analog-tools/session`

**Routes** (exposed via middleware):

- `POST /login` - Initiates OAuth flow
- `GET /callback` - OAuth provider callback handler
- `POST /logout` - Revokes tokens and clears session
- `GET /authenticated` - Checks auth status
- `GET /user` - Returns current user

**Security:**

- Tokens stored server-side only (never exposed to client)
- HMAC-SHA256 signed session cookies
- Token refresh with 5-minute safety margin
- CSRF protection via OAuth state parameter
- Configurable unprotected routes with wildcard support

**Configuration** (`AnalogAuthConfig`):

```typescript
{
  issuer: string;              // OAuth provider URL
  clientId: string;
  clientSecret: string;
  scope: string;               // e.g., "openid profile email"
  callbackUri: string;
  unprotectedRoutes?: string[]; // ['/', '/login', '/api/public/*']
  whitelistFileTypes?: string[]; // ['.css', '.js', '.png']
  sessionStorage: {
    driver: { type: 'redis' | 'cloudflare-kv-binding' | 'fs' | 'memory' }
  }
}
```

### Session Management (Functional API)

Location: `/packages/session/src`

Functional API redesigned in v0.0.6 for simplicity:

```typescript
// Initialize session
await useSession(event, {
  store,           // unstorage Storage instance
  secret,          // String or array for rotation
  name,            // Cookie name
  maxAge,          // TTL in seconds
  cookie: { ... }  // Cookie options
});

// API functions
getSession<T>(event): T | null
updateSession<T>(event, updater: (current) => Partial<T>)
destroySession(event)
regenerateSession(event)
```

**Storage:**

- Direct unstorage integration (no wrapper abstractions)
- Use `createUnstorageStore(driver)` factory
- Supports: Redis, Cloudflare KV, file system, MongoDB, etc.
- Memory driver currently has issues

### Angular Client Integration

Location: `/packages/auth-angular/src`

Uses Angular 19+ signals and modern patterns:

- `AuthService` - Provides reactive auth state
  - `isAuthenticated()` signal
  - `user()` signal with automatic type transformation
  - `login(targetUrl)` and `logout()` methods
- `authGuard` - Route guard for protected routes
- `roleGuard` - Route guard for role-based access

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MrBacony/analog-tools](https://github.com/MrBacony/analog-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
