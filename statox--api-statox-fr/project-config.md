---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal TypeScript/Express API backend serving multiple features (HomeTracker, Chords, Clipboard, Cookbook, WebWatcher, etc.) with type-safe routes, JSON schema validation, and an auto-generated SDK for the frontend.

## Development Commands

### Local Development

These commands are not for you to run, the user should have started them already:

```bash
npm run env              # Start docker environment (MySQL, LocalStack S3)
./src/tools/init-db.sh   # Initialize database tables
npm run watch            # TypeScript watcher
npm run serve            # Start server on port 3000 (watches dist/)
```

### Testing

```bash
npm run tests                    # Run route & periodic task tests
npm run tests:framework          # Run framework tests
npm run tests:packages           # Run package tests
npm run tests:all                # Run all test suites
```

To run a single test, use the npm script which invokes mocha. Use the `-f` option to filter on the name of the test.

```bash
npm run tests -- -f 'personalTracker/getAll'
```

Adding `debug=true` before the test command trigger more verbose logs useful for debugging:

```bash
debug=true npm run tests -- -f 'personalTracker'
```

### Code Quality

```bash
npm run check                    # Run lint + prettier
npm run lint                     # ESLint
npm run lint:fix                 # Fix ESLint issues
npm run prettier                 # Check formatting
npm run prettier:fix             # Fix formatting
```

### SDK Generation

```bash
npm run generate:sdk             # Generate TypeScript SDK from routes
```

The SDK is generated from route definitions and synced to the frontend codebase.

### Deployment

This must be run exclusively by the user directly on their terminal.

```bash
npm run heroku:deploy            # Deploy to production (runs checks + tests)
npm run heroku:deploy:skip-tests # Deploy without running tests
npm run heroku:ssh               # SSH into production
```

### User Management

```bash
npm run user:create              # Create local user interactively
```

## Architecture

### Route System

Routes are the core of the API. Each route is defined as a strongly-typed object in `src/libs/routes/[module]/[endpoint].ts`:

```typescript
export const route: GetRoute<InputType, OutputType> = {
    method: 'get' | 'post',
    path: '/module/endpoint',
    handler: async ({ input, loggableContext, authenticatedUser }) => { ... },
    authentication: 'none' | 'apikey-iot' | 'user2',
    scope: 'public' | 'admin' | 'homeTracker' | 'personalTracker', // Required for user2 auth
    inputSchema: { ... },      // JSON schema (POST only)
    outputSchema: { ... }      // JSON schema
};
```

Routes are registered in `src/libs/routes/index.ts` by importing and adding them to the `routes.list` array. The route system automatically:

- Validates input against JSON schema (POST routes)
- Applies authentication middleware based on `authentication` field
- Validates user scopes (for `user2` authentication)
- Handles errors via `errorHandler` middleware
- Generates OpenAPI definitions
- Powers SDK generation

### Authentication

Three authentication modes:

1. **none**: Public endpoints
2. **apikey-iot**: IoT devices using API key headers (validated by `validateAPIKeyHeader`)
3. **user2**: Session-based auth using PassportJS with local strategy
    - Login: `/auth/login` validates credentials and creates session
    - Session stored in MySQL `sessions` table
    - Cookie contains session ID + signature (validated with session secret)
    - Subsequent requests validated via `validatePassportSession` middleware
    - Scopes enforced via `validateEndpointScope` middleware

See `DEV.md` for detailed auth flow.

### Middleware Pipeline

Each route goes through middleware in `src/app.ts` (lines 72-108):

1. `loggingHandler` - Request logging
2. `multipartHandler` - File upload handling
3. Authentication middleware (based on route.authentication)
4. Input validation (POST routes only, via express-json-validator-middleware)
5. `apiPipeline` - Main route handler wrapper
6. `errorHandler` - Global error handler

### Directory Structure

- **src/libs/routes/**: Route definitions organized by module (auth, chords, clipboard, cookbook, gravitrips, health, homeTracker, personalTracker, reactor, webWatcher)
- **src/libs/middleware/**: Express middleware (auth, errors, logging, multipart, apiPipeline)
- **src/libs/modules/**: Business logic modules (ajv, auth, chords, clipboard, cookbook, ephemerides, gravitrips, homeTracker, logging, meteofrance, notifier, openapi, personalTracker, reactor, s3files, webWatcher)
- **src/libs/databases/**: Database connections (MySQL, Elasticsearch, S3)
- **src/libs/PeriodicTasks/**: Background tasks started in production
- **src/packages/**: Reusable packages (config, suncalc)
- **src/tools/**: CLI utilities (DB init, user creation, ELK tools, OpenAPI generation)
- **tests/**: Test files mirroring src structure (routes, framework, helpers, periodicTasks)

### Configuration

Configuration is managed by the `src/packages/config/` package:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/statox) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
